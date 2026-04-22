# PRD — Cálculo de Margem de Rentabilidade e Rollout Segmentado do Cartão Alfa

**Hipótese de origem:** hipotese_cartao_margem.md
**Squad:** Squad Cartões · **PM/PO:** Ana Beatriz Torres · **TL:** Bruno Ferreira
**Data:** 21/04/2026 · **Status:** Em review

---

## 1. Objetivo e contexto

Construir um motor de cálculo de margem de rentabilidade projetada por cliente para o produto Cartão Alfa e utilizá-lo como critério de priorização no rollout por segmento. O banco emite hoje cartões sem visibilidade de rentabilidade individual, o que resulta em 23% da carteira com margem negativa. O objetivo é ativar primeiro os clientes com maior margem projetada, melhorando o resultado do produto em até 18%.

---

## 2. Personas, usuários ou processos afetados

- **Gestores de Produto (Cartões):** passam a visualizar margem projetada por segmento antes de acionar rollout
- **Time Comercial:** recebe lista priorizada de clientes elegíveis por segmento de margem
- **Time de Risco:** valida e monitora as premissas do modelo de margem
- **Clientes:** recebem oferta do cartão conforme enquadramento no segmento de rollout

---

## 3. Escopo

| Funcionalidade / comportamento | Prioridade | Justificativa |
|---|---|---|
| Cálculo de margem projetada por cliente (12 meses) | Must have | Base do produto inteiro |
| Classificação de clientes em segmentos de margem (Premium, Padrão, Risco) | Must have | Critério do rollout |
| API de consulta de margem por CPF | Must have | Integração com motor de ofertas |
| Painel de acompanhamento de margem por segmento (squad interna) | Should have | Monitoramento operacional |
| Exportação de listas de rollout por segmento | Must have | Acionar campanha comercial |
| Retroalimentação real vs. projetado após 30/60/90 dias | Should have | Melhoria contínua do modelo |
| Modelo de margem para novos clientes (sem histórico) | Won't have | Escopo futuro — sprint seguinte |

---

## 4. Fluxo funcional

1. O sistema coleta dados do cliente nos sistemas de origem (CORE_BANK, TXN_SYSTEM, RISK_ENGINE, ACTU_SYSTEM, COST_ALLOC, FRAUD_SYS)
2. O motor de margem calcula a margem projetada mensal por cliente com base nas regras definidas abaixo
3. O cliente é classificado em um segmento de margem (Premium / Padrão / Risco)
4. A classificação é persistida e disponibilizada via API e via exportação
5. O time comercial consome a lista priorizada para disparar o rollout por fase
6. Após 30/60/90 dias, o sistema compara margem realizada vs. projetada e retroalimenta o modelo

---

## 5. Critérios de aceite de negócio

| Dado que... / Quando... / Então... | Tipo |
|---|---|
| Dado que o cliente tem histórico de 6 meses de transações no TXN_SYSTEM, quando o motor de margem rodar, então deve calcular a margem projetada com base nas 6 regras de receita e 4 de custo definidas no PRD. | Happy path |
| Dado que o cliente não tem histórico de transações, quando o motor rodar, então deve usar a mediana do segmento de renda como proxy de gasto projetado. | Edge case |
| Dado que o motor calculou a margem, quando o resultado for > R$ 60/mês, então o cliente é classificado como Segmento Premium. | Happy path |
| Dado que o motor calculou a margem entre R$ 20 e R$ 60/mês, então o cliente é classificado como Segmento Padrão. | Happy path |
| Dado que o motor calculou a margem < R$ 20/mês ou negativa, então o cliente é classificado como Segmento Risco e não entra nas fases 1 e 2 do rollout. | Happy path |
| Dado que um sistema de origem está indisponível, quando o motor tentar coletar dados, então deve registrar a falha, usar o último valor processado e não recalcular a margem com dado parcial. | Edge case |
| Dado que o rollout da fase 1 está ativo, quando uma lista for exportada, então deve conter apenas clientes Segmento Premium sem cartão ativo. | Happy path |

---

## 6. Métricas de sucesso

- **Primária:** margem líquida média mensal por cartão ativo — baseline R$ 18,40 → meta R$ 21,70
- **Secundária:** % de cartões com margem negativa — baseline 23% → meta < 12%
- **Guardrail:** taxa de aprovação no rollout ≥ 35%
- **Janela de medição:** 90 dias após go-live da fase 1

---

## 7. Origem funcional dos dados

| Informação necessária | Sistema origem | Tabela / arquivo / entidade | Campo(s) relevantes | Chave de referência | Dono da informação | Observações |
|---|---|---|---|---|---|---|
| Volume de gastos mensais (12 meses) | TXN_SYSTEM | tb_transacao_cartao | vl_transacao, dt_transacao, cd_tipo_transacao | nr_cpf | Engenharia de Dados | Filtrar apenas transações de compra (cd_tipo = 'COMPRA') |
| Taxa de intercâmbio (MDR) por transação | TXN_SYSTEM | tb_transacao_cartao | vl_mdr, cd_bandeira | nr_cpf + id_transacao | Engenharia de Dados | MDR já está calculado por transação |
| Limite de crédito aprovado | CORE_BANK | tb_limite_credito | vl_limite_aprovado, vl_limite_disponivel | nr_cpf | Core Banking | Usar limite aprovado, não disponível |
| Saldo devedor médio (12 meses) | CORE_BANK | tb_fatura_cartao | vl_saldo_devedor, dt_referencia | nr_cpf | Core Banking | Base para cálculo de juros do rotativo |
| Taxa de PDD projetada por score | RISK_ENGINE | tb_pdd_projecao | pc_pdd_projetado, nr_score_credito, cd_faixa_risco | nr_cpf | Risco de Crédito (Rafael Gomes) | Revisar regra de faixa de risco com Rafael |
| Receita de seguro prestamista | ACTU_SYSTEM | tb_premio_seguro | vl_premio_mensal, dt_vigencia | nr_cpf + id_apolice | Atuarial (Marcelo Souza) | Usar prêmio líquido de sinistro |
| Custo operacional por cartão | COST_ALLOC | tb_custo_produto | vl_custo_unit_mensal, cd_produto | cd_produto = 'CARTAO_ALFA' | Finanças (Camila Neves) | Custo é fixo por produto, não por cliente — R$ 9,20/cartão/mês confirmado |
| Taxa histórica de fraude por segmento | FRAUD_SYS | tb_fraude_historico | pc_fraude, cd_segmento_risco, dt_referencia | cd_segmento_risco | Prevenção a Fraudes | Usar média dos últimos 12 meses por segmento de risco |

---

## 8. Regras de negócio detalhadas com base em dados

| Regra | Descrição funcional | Dados necessários | Fonte funcional | Responsável pela validação |
|---|---|---|---|---|
| RN01 — Receita de intercâmbio | Receita de intercâmbio = soma do vl_mdr de todas as transações de compra dos últimos 12 meses, dividido por 12 (média mensal) | vl_mdr, cd_tipo_transacao | TXN_SYSTEM | Engenharia de Dados |
| RN02 — Receita de juros do rotativo | Receita de juros = saldo devedor médio × 12,5% a.m. (taxa spread do produto). Aplicar apenas se vl_saldo_devedor > 0 | vl_saldo_devedor, dt_referencia | CORE_BANK | Risco de Crédito (Rafael Gomes) |
| RN03 — Receita de anuidade | Receita de anuidade = R$ 240/ano ÷ 12 = R$ 20/mês fixo por cliente com cartão ativo | Fixo do produto | Produto (Ana Torres) | Ana Torres |
| RN04 — Receita de seguro | Receita de seguro = vl_premio_mensal líquido de sinistro do ACTU_SYSTEM | vl_premio_mensal | ACTU_SYSTEM | Atuarial (Marcelo Souza) |
| RN05 — Custo de PDD | Custo de PDD = vl_limite_aprovado × pc_pdd_projetado. A faixa de PDD é determinada pelo nr_score_credito: score ≥ 700 → 2,1%; score 620-699 → 4,8%; score 500-619 → 9,3%; score < 500 → fora do produto | vl_limite_aprovado, nr_score_credito, pc_pdd_projetado | RISK_ENGINE | Risco de Crédito (Rafael Gomes) |
| RN06 — Custo de fraude | Custo de fraude = volume de gastos projetado × taxa histórica de fraude do segmento de risco do cliente | pc_fraude, cd_segmento_risco | FRAUD_SYS | Prevenção a Fraudes |
| RN07 — Custo de captação | Custo de captação = saldo devedor médio × (CDI + 3,2% a.a.) ÷ 12. CDI de referência atualizado mensalmente via parâmetro de sistema | vl_saldo_devedor | CORE_BANK / Parâmetro | Finanças (Camila Neves) |
| RN08 — Custo operacional | Custo operacional = R$ 9,20/mês fixo por cartão ativo (custo alocado do produto) | Fixo do produto | Finanças (Camila Neves) | Camila Neves |
| RN09 — Margem líquida | Margem líquida = (RN01 + RN02 + RN03 + RN04) − (RN05 + RN06 + RN07 + RN08) | Todas as anteriores | Produto | Ana Torres |
| RN10 — Segmentação | Segmento Premium: margem ≥ R$ 60/mês. Segmento Padrão: R$ 20 ≤ margem < R$ 60. Segmento Risco: margem < R$ 20 | Resultado de RN09 | Produto | Ana Torres |
| RN11 — Proxy sem histórico | Cliente sem histórico de 6 meses usa como proxy de gasto mensal a mediana do gasto dos clientes com mesmo cd_segmento_risco e mesma faixa de renda cadastrada no CORE_BANK | cd_segmento_risco, faixa_renda | CORE_BANK / RISK_ENGINE | Risco de Crédito (Rafael Gomes) |

---

## 9. Mapeamento AS IS funcional

- **Processo atual:** O motor de ofertas consulta apenas score de crédito e renda mínima para decidir elegibilidade. Não há cálculo de rentabilidade individual.
- **Sistemas envolvidos:** CORE_BANK (elegibilidade), RISK_ENGINE (score)
- **Entradas:** nr_cpf do cliente, cd_produto
- **Saídas:** flag de elegibilidade (S/N)
- **Regras conhecidas no legado:** score ≥ 500 + renda ≥ R$ 1.500 = elegível
- **Pontos não esclarecidos:**
  - Existe alguma regra de exclusão por negativação cadastral no CORE_BANK que precisa ser mantida?
  - O ACTU_SYSTEM alimenta em batch ou real-time?

---

## 10. Dicionário funcional mínimo

| Campo | Significado | Regra associada | Obrigatório? | Exemplo |
|---|---|---|---|---|
| vl_margem_projetada | Margem líquida mensal projetada do cliente com o produto | RN09 | Sim | R$ 47,30 |
| cd_segmento_margem | Segmento de rentabilidade do cliente | RN10 | Sim | PREMIUM / PADRAO / RISCO |
| vl_receita_total | Soma de todas as receitas projetadas (RN01 a RN04) | RN01-04 | Sim | R$ 68,90 |
| vl_custo_total | Soma de todos os custos projetados (RN05 a RN08) | RN05-08 | Sim | R$ 21,60 |
| dt_calculo | Data em que a margem foi calculada | — | Sim | 2026-04-20 |
| fl_historico_disponivel | Indica se o cliente tinha ≥ 6 meses de histórico no momento do cálculo | RN11 | Sim | S / N |

---

## 11. Dependências externas

- **Outros times:** Risco de Crédito (validação RN05, RN11), Finanças (validação RN07, RN08), Atuarial (validação RN04), Prevenção a Fraudes (validação RN06)
- **APIs / serviços / sistemas:** TXN_SYSTEM, CORE_BANK, RISK_ENGINE, ACTU_SYSTEM, COST_ALLOC, FRAUD_SYS
- **Prazo externo:** Campanha comercial de rollout fase 1 prevista para 02/06/2026 — motor precisa estar pronto até 26/05/2026

---

## 12. Requisitos não-funcionais (visão de negócio)

- O cálculo deve ser reprocessável: em caso de falha em sistema de origem, deve ser possível reprocessar apenas o cliente afetado sem recalcular toda a base
- Rastreabilidade obrigatória: cada cálculo deve registrar os valores de entrada usados por regra, para auditoria de Risco e Finanças
- O CDI de referência (RN07) deve ser parametrizável sem deploy — ajuste mensal pela equipe de Finanças
- A segmentação deve ser atualizável: um cliente pode mudar de segmento a cada ciclo de recálculo (mensal)

---

## 13. Perguntas em aberto

- [ ] Existe alguma regra de exclusão por negativação no CORE_BANK que precisa ser preservada no novo fluxo? — responsável: Rafael Gomes — prazo: 24/04/2026
- [ ] O ACTU_SYSTEM entrega o prêmio líquido de sinistro ou bruto? — responsável: Marcelo Souza — prazo: 24/04/2026
- [ ] O recálculo da margem será mensal (batch) ou sob demanda (API)? — responsável: Bruno Ferreira (TL) — prazo: 28/04/2026

---

## Gate de handoff para o Tech Lead

- [x] Escopo Must/Should/Won't acordado com stakeholders?
- [x] Critérios de aceite cobrem happy paths e edge cases principais?
- [x] Fluxo funcional disponível?
- [x] Dependências externas mapeadas?
- [x] Origem funcional dos dados identificada?
- [x] Tabelas / arquivos / entidades de referência informados?
- [x] Campos relevantes para a regra documentados?
- [x] Donos das informações e regras identificados?
- [ ] Perguntas em aberto resolvidas? (3 pendentes — ver seção 13)
- [x] TL consegue começar a solução sem precisar fazer discovery funcional no lugar do PO?