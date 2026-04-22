# Tech Spec / RFC — Motor de Margem de Rentabilidade — Cartão Alfa

**PRD de origem:** prd_cartao_margem.md
**Tech Lead:** Bruno Ferreira · **Revisores:** Thiago Lima (Arquitetura), Carla Dias (Dados), Paulo Vasconcelos (SRE)
**Data:** 28/04/2026 · **Status:** Em review · **Versão:** v1

---

## 1. Resumo técnico (TL;DR)

Construção de um pipeline de cálculo de margem de rentabilidade projetada por cliente para o Cartão Alfa. O sistema coleta dados de 6 origens distintas, aplica as 11 regras de negócio definidas no PRD, persiste o resultado numa tabela analítica e disponibiliza via API REST para o motor de ofertas e via exportação CSV para o time comercial. O processamento principal é batch mensal com suporte a reprocessamento individual por CPF.

---

## 2. Contexto técnico atual

Hoje o motor de ofertas (OFFER_ENGINE) consulta apenas dois parâmetros: score de crédito (RISK_ENGINE) e renda mínima (CORE_BANK). Não há camada de rentabilidade no fluxo. A elegibilidade é binária (S/N) e não considera margem projetada.

Componentes envolvidos atualmente:
- OFFER_ENGINE — serviço Node.js que decide elegibilidade
- RISK_ENGINE — API REST interna (score + PDD)
- CORE_BANK — banco Oracle legado, acesso via JDBC e views materializadas

---

## 3. Insumos recebidos do PRD

- Sistemas origem identificados: sim (TXN_SYSTEM, CORE_BANK, RISK_ENGINE, ACTU_SYSTEM, COST_ALLOC, FRAUD_SYS)
- Tabelas informadas: tb_transacao_cartao, tb_limite_credito, tb_fatura_cartao, tb_pdd_projecao, tb_premio_seguro, tb_custo_produto, tb_fraude_historico
- Campos mapeados funcionalmente: sim — ver seção 7 do PRD
- Regras validadas pelo negócio: RN01 a RN11 validadas. RN05 (faixas de PDD) e RN11 (proxy sem histórico) ainda aguardam confirmação final de Rafael Gomes
- Lacunas remanescentes: (1) regra de exclusão por negativação; (2) prêmio líquido vs. bruto no ACTU_SYSTEM; (3) decisão de batch vs. on-demand — assumimos batch mensal + reprocessamento individual sob demanda

---

## 4. Solução proposta

### Novos componentes

- **margin-calculator** (Python / Spark job): pipeline de cálculo batch que lê das origens, aplica RN01-RN11 e grava em tb_margem_cliente
- **margin-api** (FastAPI): expõe endpoint GET /v1/margem/{cpf} para consulta pelo OFFER_ENGINE e outros consumers
- **margin-export-job** (Python script): gera CSV de clientes por segmento para uso comercial
- **margin-params-service** (microsserviço leve): persiste parâmetros ajustáveis (CDI, faixas de segmento) sem necessidade de deploy

### Mudanças em existentes

- **OFFER_ENGINE**: incluir chamada ao margin-api antes de emitir elegibilidade; incluir cd_segmento_margem na resposta de oferta

### Fluxo de dados

1. Job batch roda todo dia 1 do mês (00h) via Airflow
2. Coleta dados das 6 origens em paralelo (Spark read)
3. Aplica RN01-RN11 por cliente
4. Persiste resultado em tb_margem_cliente (Redshift)
5. OFFER_ENGINE consulta margin-api em tempo real por CPF
6. margin-export-job gera arquivo por segmento ao fim do processamento batch

---

## 5. Alternativas consideradas

| Alternativa | Status | Por que descartada / escolhida |
|---|---|---|
| Cálculo em tempo real (on-demand apenas) | Descartada | Latência inaceitável para consultar 6 sistemas ao mesmo tempo na jornada do cliente |
| Cálculo batch + cache Redis | Escolhida | Melhor equilíbrio entre latência na consulta e consistência dos dados |
| Motor de regras externo (Drools/Camunda) | Descartada | Overkill para 11 regras estáticas; adiciona dependência operacional desnecessária |
| Implementar em dbt puro | Descartada | ACTU_SYSTEM e FRAUD_SYS não têm conector dbt aprovado; exigiria Python de qualquer forma |

---

## 6. Modelo de dados e contratos

### Schema

```sql
CREATE TABLE tb_margem_cliente (
    nr_cpf           VARCHAR(11)    NOT NULL,
    dt_calculo       DATE           NOT NULL,
    vl_receita_total NUMERIC(12,2)  NOT NULL,
    vl_custo_total   NUMERIC(12,2)  NOT NULL,
    vl_margem_projetada NUMERIC(12,2) NOT NULL,
    cd_segmento_margem VARCHAR(10)  NOT NULL,  -- PREMIUM | PADRAO | RISCO
    fl_historico_disponivel CHAR(1) NOT NULL,  -- S | N
    vl_receita_intercambio NUMERIC(12,2),
    vl_receita_juros       NUMERIC(12,2),
    vl_receita_anuidade    NUMERIC(12,2),
    vl_receita_seguro      NUMERIC(12,2),
    vl_custo_pdd           NUMERIC(12,2),
    vl_custo_fraude        NUMERIC(12,2),
    vl_custo_captacao      NUMERIC(12,2),
    vl_custo_operacional   NUMERIC(12,2),
    dt_processamento TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (nr_cpf, dt_calculo)
);
```

### API

```text
GET /v1/margem/{cpf}
Response 200: {
  "cpf": "...",
  "dt_calculo": "2026-05-01",
  "vl_margem_projetada": 47.30,
  "cd_segmento_margem": "PADRAO",
  "fl_historico_disponivel": "S"
}
Response 404: { "error": "cliente_sem_calculo" }
Response 422: { "error": "cpf_invalido" }

POST /v1/margem/reprocessar
Body: { "cpf": "..." }
Response 202: { "job_id": "...", "status": "enqueued" }
```

### Eventos / mensagens
- Ao fim do batch mensal: evento `margem.calculada.concluida` publicado no tópico Kafka `cartao-margem-eventos` com contagem por segmento

---

## 7. Requisitos não-funcionais

- **Performance:** GET /v1/margem/{cpf} com p99 < 80ms (leitura de cache Redis)
- **Disponibilidade:** margin-api com SLA de 99,5% (não é crítico em tempo real, apenas na jornada de oferta)
- **Escalabilidade:** batch precisa processar ~480 mil clientes em janela de 4 horas (até 06h do dia 1)
- **Segurança:** CPF deve ser mascarado nos logs; acesso à margin-api restrito via mTLS entre serviços internos
- **Auditoria:** cada linha da tb_margem_cliente registra os valores de cada componente de receita e custo individualmente (rastreabilidade por RN)

---

## 8. Estratégia de rollout

- **Feature flag:** sim — flag `margem_cartao_enabled` por segmento. Fase 1: habilitar PREMIUM apenas. Fase 2: PADRAO. Fase 3: RISCO (se decidido)
- **Migração de dados:** sem migração. Primeiro batch gera o estado inicial para todos os clientes.
- **Rollback:** desabilitar feature flag no OFFER_ENGINE reverte para comportamento atual (score + renda). tb_margem_cliente não é excluída.
- **Ordem de deploy:** (1) margin-params-service → (2) margin-calculator (batch) → (3) margin-api + cache Redis → (4) margin-export-job → (5) OFFER_ENGINE com flag desabilitada → (6) habilitar flag PREMIUM após validação do batch

---

## 9. Riscos técnicos

| Risco | Severidade | Mitigação |
|---|---|---|
| ACTU_SYSTEM não tem SLA de disponibilidade definido — batch pode falhar | Alta | Implementar fallback: se ACTU_SYSTEM indisponível, usar último valor processado do cliente; logar e alertar |
| CORE_BANK legado pode ter lentidão em leitura de fatura para 480k clientes | Alta | Usar view materializada existente + leitura paralela com Spark JDBC particionado por faixa de CPF |
| CDI de referência atualizado fora do prazo pela equipe de Finanças impacta cálculo de RN07 | Média | Alertar automaticamente se CDI não for atualizado até dia 28 do mês anterior |
| Divergência entre prêmio bruto/líquido do ACTU_SYSTEM (lacuna do PRD ainda aberta) | Média | Aguardar resposta de Marcelo Souza; implementar parâmetro fl_premio_liquido para ajuste sem redeploy |

---

## 10. Plano de observabilidade

- **Métricas:** total de clientes calculados por batch, % por segmento (PREMIUM/PADRAO/RISCO), clientes sem histórico (fl_N), falhas por sistema de origem, latência da margin-api (p50/p95/p99)
- **Logs:** início e fim de cada batch com duração; falhas de coleta por sistema de origem; reprocessamentos manuais com CPF mascarado
- **Alertas:**
  - Batch não finalizado até 06h → PagerDuty (P1)
  - % PREMIUM < 10% ou > 60% após batch → alerta para PO + Risco (possível erro de regra)
  - margin-api com error rate > 1% em 5min → alerta SRE
  - CDI não atualizado até dia 28 → alerta Finanças

---

## 11. Lacunas assumidas pelo técnico

| Item | Impacto | Ação tomada | Risco |
|---|---|---|---|
| Prêmio líquido vs. bruto no ACTU_SYSTEM | Médio | Implementar parâmetro fl_premio_liquido ajustável sem deploy | Se errado, receita de seguro superestimada |
| Regra de exclusão por negativação no CORE_BANK | Médio | Mantida a regra atual de exclusão do OFFER_ENGINE (score < 500) até confirmação de Rafael | Pode incluir cliente que deveria ser excluído |
| Recálculo sob demanda: SLA de resposta do POST /reprocessar | Baixo | Assumido processamento assíncrono com retorno em até 30 min | Impacto apenas em casos de suporte |

---

## 12. Proposta de quebra em histórias

| História proposta | Tipo | Dependência |
|---|---|---|
| Criar tabela tb_margem_cliente e pipeline de ingestão dos sistemas de origem | Dados / Infra | — |
| Implementar RN01-RN04 (componentes de receita) no margin-calculator | Backend | Depende da história de ingestão |
| Implementar RN05-RN08 (componentes de custo) no margin-calculator | Backend | Depende da história de ingestão |
| Implementar RN09 (margem líquida) e RN10 (segmentação) | Backend | Depende das histórias de receita e custo |
| Implementar RN11 (proxy sem histórico) | Backend | Depende de RN09/RN10 |
| Criar margin-params-service (CDI e faixas parametrizáveis) | Backend | — |
| Criar margin-api (GET /v1/margem/{cpf} + cache Redis) | Backend | Depende de RN09/RN10 |
| Criar POST /v1/margem/reprocessar (reprocessamento individual) | Backend | Depende da margin-api |
| Integrar OFFER_ENGINE com margin-api + feature flag por segmento | Backend | Depende da margin-api |
| Criar margin-export-job (CSV por segmento para comercial) | Backend / Dados | Depende de RN10 |
| Criar painel de acompanhamento de margem por segmento (squad) | Frontend / Dados | Depende da tabela tb_margem_cliente |
| Implementar observabilidade (métricas, logs, alertas) | Infra / Obs. | Depende de todas as histórias de backend |

---

## 13. Estimativa de esforço

| Área | Complexidade | Observação |
|---|---|---|
| Backend (motor de margem) | L (12 dias) | 6 integrações + 11 regras + reprocessamento |
| Backend (margin-api + OFFER_ENGINE) | M (4 dias) | API simples + cache Redis |
| Dados / Infra (pipeline + schema) | M (5 dias) | Airflow job + Redshift + views |
| Dados (export job) | S (2 dias) | CSV simples por segmento |
| Frontend (painel squad) | M (3 dias) | Dashboard básico |
| QA / testes | M (5 dias) | Testes de regra + integração + batch |

---

## 14. Perguntas em aberto

- [ ] Prêmio líquido vs. bruto no ACTU_SYSTEM — responsável: Marcelo Souza — prazo: 30/04/2026
- [ ] Regra de exclusão por negativação no CORE_BANK — responsável: Rafael Gomes — prazo: 30/04/2026

---

## Gate de refinamento — TL + time + PM/PO

- [x] Time entende a solução sem depender exclusivamente do TL?
- [x] Alternativas descartadas documentadas com justificativa?
- [x] Riscos com mitigação definida?
- [x] Ordem de deploy e dependências claras?
- [x] Estratégia de rollback existe?
- [x] Proposta de histórias é quebrável em itens de sprint?
- [x] Plano de observabilidade definido antes de codar?
- [x] PM/PO tem insumo suficiente para criar as histórias finais no board?

---

# Histórias — Motor de Margem de Rentabilidade — Cartão Alfa

**Origem:** prd_cartao_margem.md / tech_spec_cartao_margem.md
**Responsável pela criação no board:** Ana Beatriz Torres (PO)

---

## H-01 — Pipeline de ingestão dos sistemas de origem

**Título:** Criar pipeline de ingestão dos dados de margem dos sistemas de origem

Como squad de Cartões
Quero coletar e consolidar os dados de TXN_SYSTEM, CORE_BANK, RISK_ENGINE, ACTU_SYSTEM, COST_ALLOC e FRAUD_SYS
Para ter insumos disponíveis no Redshift para o cálculo de margem por cliente

**Critérios de aceite**
- CA1: Dados de todas as 6 origens disponíveis na staging area do Redshift após execução do job
- CA2: Campos coletados correspondem exatamente aos mapeados na seção 7 do PRD
- CA3: Falha em sistema de origem individual não interrompe coleta dos demais — falha é registrada e alertada
- CA4: Job é idempotente: reexecutar para a mesma data não duplica registros

**Dependências:** Acesso liberado às 6 origens (confirmar com Infra)
**Sizing:** M
**Tasks:**
- Configurar conexões JDBC (CORE_BANK) e APIs REST (RISK_ENGINE, ACTU_SYSTEM, FRAUD_SYS)
- Criar tabelas staging no Redshift
- Criar Airflow DAG com tasks paralelas por sistema de origem
- Implementar lógica de fallback com último valor conhecido
- Testes de integração com dados reais (ambiente homologação)

---

## H-02 — Cálculo dos componentes de receita (RN01 a RN04)

**Título:** Implementar cálculo dos componentes de receita da margem (intercâmbio, juros, anuidade, seguro)

Como motor de margem
Quero calcular a receita projetada mensal por cliente
Para compor a margem bruta antes da dedução dos custos

**Critérios de aceite**
- CA1: RN01 (intercâmbio) = média mensal de vl_mdr das transações de compra dos últimos 12 meses
- CA2: RN02 (juros rotativo) = vl_saldo_devedor médio × 12,5% a.m., aplicado apenas quando saldo > 0
- CA3: RN03 (anuidade) = R$ 20,00 fixo para todo cliente ativo
- CA4: RN04 (seguro) = vl_premio_mensal do ACTU_SYSTEM (líquido ou bruto conforme parâmetro fl_premio_liquido)
- CA5: vl_receita_total = soma de RN01 a RN04, persistida em tb_margem_cliente

**Dependências:** H-01 concluída
**Sizing:** M
**Tasks:**
- Implementar RN01 no margin-calculator com filtro cd_tipo_transacao = 'COMPRA'
- Implementar RN02 com validação de saldo > 0
- Implementar RN03 como constante parametrizável
- Implementar RN04 com leitura do parâmetro fl_premio_liquido
- Testes unitários por regra com casos limite (saldo zero, sem transações)

---

## H-03 — Cálculo dos componentes de custo (RN05 a RN08)

**Título:** Implementar cálculo dos componentes de custo da margem (PDD, fraude, captação, operacional)

Como motor de margem
Quero calcular o custo projetado mensal por cliente
Para deduzir da receita bruta e obter a margem líquida

**Critérios de aceite**
- CA1: RN05 (PDD): score ≥ 700 → 2,1%; 620-699 → 4,8%; 500-619 → 9,3%; < 500 → cliente excluído do cálculo
- CA2: RN06 (fraude): volume de gastos projetado × taxa histórica do cd_segmento_risco no FRAUD_SYS
- CA3: RN07 (captação): saldo devedor médio × (CDI_referencia + 3,2%) ÷ 12; CDI lido do margin-params-service
- CA4: RN08 (operacional): R$ 9,20 fixo por cliente
- CA5: vl_custo_total = soma de RN05 a RN08, persistida em tb_margem_cliente

**Dependências:** H-01 concluída; margin-params-service disponível (H-06)
**Sizing:** M
**Tasks:**
- Implementar RN05 com tabela de faixas de score e PDD
- Implementar RN06 com join de segmento de risco x taxa de fraude histórica
- Implementar RN07 com leitura dinâmica do CDI via margin-params-service
- Implementar RN08 como constante parametrizável
- Testes unitários com clientes em cada faixa de score

---

## H-04 — Margem líquida e segmentação (RN09 e RN10)

**Título:** Implementar cálculo da margem líquida e classificação por segmento

Como motor de margem
Quero calcular a margem líquida por cliente e classificar em PREMIUM, PADRAO ou RISCO
Para alimentar o motor de ofertas com o segmento correto para o rollout

**Critérios de aceite**
- CA1: vl_margem_projetada = vl_receita_total − vl_custo_total
- CA2: PREMIUM quando vl_margem_projetada ≥ 60,00
- CA3: PADRAO quando 20,00 ≤ vl_margem_projetada < 60,00
- CA4: RISCO quando vl_margem_projetada < 20,00
- CA5: cd_segmento_margem e vl_margem_projetada persistidos em tb_margem_cliente com dt_calculo
- CA6: cliente com score < 500 não é calculado — registro ignorado com log de exclusão

**Dependências:** H-02 e H-03 concluídas
**Sizing:** P
**Tasks:**
- Implementar RN09 somando receitas e subtraindo custos
- Implementar RN10 com lógica de segmentação
- Implementar exclusão de clientes com score < 500 com log
- Testes de borda: margem exatamente em R$ 20 e R$ 60

---

## H-05 — Proxy sem histórico (RN11)

**Título:** Implementar proxy de gasto para clientes sem 6 meses de histórico

Como motor de margem
Quero calcular margem de clientes sem histórico suficiente usando a mediana do segmento
Para não excluir clientes novos do cálculo de elegibilidade

**Critérios de aceite**
- CA1: Clientes com menos de 6 meses de histórico recebem fl_historico_disponivel = 'N'
- CA2: Proxy de gasto = mediana do gasto mensal de clientes com mesmo cd_segmento_risco e mesma faixa de renda no CORE_BANK
- CA3: Proxy é calculado uma vez por batch e reutilizado para todos os clientes sem histórico do mesmo grupo
- CA4: Resultado do cálculo com proxy é marcado claramente em fl_historico_disponivel = 'N' na tabela

**Dependências:** H-04 concluída; validação de Rafael Gomes (RN11)
**Sizing:** P
**Tasks:**
- Calcular mediana por segmento de risco + faixa de renda antes do loop principal
- Aplicar proxy nos clientes identificados como sem histórico
- Persistir fl_historico_disponivel
- Testes com base mista (com e sem histórico)

---

## H-06 — Parâmetros configuráveis sem deploy

**Título:** Criar margin-params-service para gestão de parâmetros sem redeploy

Como equipe de Finanças e Produto
Quero atualizar CDI de referência e faixas de segmentação sem necessidade de deploy
Para ajustar o cálculo mensalmente sem depender de engenharia

**Critérios de aceite**
- CA1: Interface (endpoint ou painel simples) permite atualizar CDI_referencia, faixas de segmento (R$ 20 e R$ 60) e fl_premio_liquido sem deploy
- CA2: margin-calculator lê os parâmetros do params-service no início de cada execução batch
- CA3: Alerta automático disparado se CDI não for atualizado até o dia 28 do mês

**Dependências:** —
**Sizing:** P
**Tasks:**
- Criar endpoint autenticado de leitura e escrita de parâmetros
- Implementar leitura dos parâmetros no início do job batch
- Configurar alerta de CDI não atualizado

---

## H-07 — API de consulta de margem por CPF

**Título:** Criar margin-api com GET /v1/margem/{cpf} e cache Redis

Como OFFER_ENGINE e consumidores internos
Quero consultar a margem projetada de um cliente por CPF em tempo real
Para usar o segmento de margem como critério de elegibilidade no motor de ofertas

**Critérios de aceite**
- CA1: GET /v1/margem/{cpf} retorna vl_margem_projetada, cd_segmento_margem, dt_calculo e fl_historico_disponivel
- CA2: CPF inválido retorna 422
- CA3: CPF sem cálculo retorna 404
- CA4: Resposta servida a partir de cache Redis com TTL de 24h
- CA5: p99 da resposta < 80ms em carga esperada

**Dependências:** H-04 concluída; Redis provisionado
**Sizing:** P
**Tasks:**
- Criar endpoint FastAPI com validação de CPF
- Configurar cache Redis com TTL
- Implementar fallback para leitura direta do Redshift se Redis indisponível
- Teste de carga (k6) simulando ~500 req/s

---

## H-08 — Integração OFFER_ENGINE + feature flag por segmento

**Título:** Integrar OFFER_ENGINE com margin-api e habilitar rollout por segmento via feature flag

Como motor de ofertas
Quero considerar o segmento de margem do cliente antes de emitir elegibilidade para o Cartão Alfa
Para fazer o rollout gradual começando pelo segmento PREMIUM

**Critérios de aceite**
- CA1: OFFER_ENGINE consulta margin-api antes de responder elegibilidade quando feature flag `margem_cartao_enabled` está ativa
- CA2: Com flag PREMIUM ativa: apenas clientes PREMIUM são elegíveis (score + renda + segmento)
- CA3: Com flag desabilitada: comportamento atual preservado (score + renda apenas)
- CA4: Rollback = desabilitar flag, sem necessidade de redeploy
- CA5: cd_segmento_margem incluído na resposta de oferta para rastreabilidade

**Dependências:** H-07 concluída
**Sizing:** P
**Tasks:**
- Implementar chamada ao margin-api no fluxo do OFFER_ENGINE
- Configurar feature flag por segmento no sistema de flags existente
- Atualizar contrato de resposta do OFFER_ENGINE
- Teste de regressão no fluxo de elegibilidade completo

---

## H-09 — Exportação de lista de rollout por segmento

**Título:** Criar job de exportação CSV com clientes por segmento para o time comercial

Como time comercial
Quero receber ao fim de cada batch uma lista de clientes por segmento de margem sem cartão ativo
Para acionar a campanha de rollout por fase

**Critérios de aceite**
- CA1: Arquivo CSV gerado automaticamente ao fim do batch com clientes PREMIUM sem cartão ativo
- CA2: Arquivo contém: CPF mascarado (primeiros 3 e últimos 2 dígitos visíveis), segmento, margem projetada, dt_calculo
- CA3: Arquivo disponível em bucket S3 com path /cartao-margem/rollout/{ano}/{mes}/segmento_premium.csv
- CA4: Notificação automática para Ana Torres (PO) ao fim da geração

**Dependências:** H-04 concluída
**Sizing:** P

---

## H-10 — Observabilidade do motor de margem

**Título:** Implementar métricas, logs e alertas do motor de margem

Como squad e SRE
Quero visibilidade completa sobre o funcionamento do batch e da API de margem
Para identificar falhas, anomalias de regra e problemas de disponibilidade rapidamente

**Critérios de aceite**
- CA1: Dashboard com: total de clientes calculados, % por segmento, % sem histórico, falhas por sistema de origem, duração do batch
- CA2: Alerta PagerDuty P1 se batch não finalizar até 06h do dia 1
- CA3: Alerta automático se % PREMIUM < 10% ou > 60% após batch (possível erro de regra)
- CA4: Alerta SRE se error rate da margin-api > 1% em janela de 5 min
- CA5: CPF nunca aparece em logs sem mascaramento

**Dependências:** H-07 concluída, batch completo (H-04)
**Sizing:** M
```

---

**Arquivos a criar:**
- hipotese_cartao_margem.md
- prd_cartao_margem.md
- tech_spec_cartao_margem.md
- historias_cartao_margem.md

**Verificação sugerida:**
1. Confirmar com Rafael Gomes as 2 lacunas ainda abertas (negativação + faixas PDD finais)
2. Confirmar com Marcelo Souza se prêmio do ACTU_SYSTEM é líquido ou bruto
3. Rodar o batch de homologação com uma amostra de 1.000 clientes antes do go-live

Quer que eu crie os arquivos no workspace ou quer ajustar algum conteúdo antes?