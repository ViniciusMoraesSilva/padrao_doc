# Documento de Hipótese — Cálculo de Margem e Rollout do Cartão Alfa

**Autor:** Ana Beatriz Torres (PO)
**Data:** 14/04/2026
**Squad:** Squad Cartões
**Status:** Em review

---

## 1. Problema observado

O Cartão Alfa hoje é ofertado de forma indiscriminada para todos os segmentos de clientes elegíveis, sem considerar a margem de rentabilidade projetada por cliente. Com isso, o banco está emitindo cartões para segmentos com alto custo de fraude, elevada inadimplência e baixo volume de transações — gerando margem líquida negativa para uma parcela da carteira.

Não existe hoje nenhum motor de cálculo de margem por cliente para o produto de cartão. A decisão de rollout é baseada apenas em score de crédito e renda mínima, sem considerar receita projetada de intercâmbio, juros, seguro e custo operacional.

---

## 2. Hipótese de solução

Se calcularmos a margem de rentabilidade projetada por cliente (receita estimada menos custos estimados) e usarmos esse indicador como critério de priorização no rollout, acreditamos que a margem média do produto vai aumentar em pelo menos 18%, porque o banco passará a ativar primeiro os clientes com maior potencial de receita e menor custo de risco.

---

## 3. Evidências e contexto

- [x] Análise interna (Dez/2025): 23% da carteira atual de cartões opera com margem líquida mensal negativa
- [x] Relatório de risco (Jan/2026): os segmentos com score entre 500-620 concentram 61% do PDD total do produto
- [x] Benchmarking setorial: bancos digitais com motor de rentabilidade por produto reportam margem 22% superior à média do setor
- [ ] Simulação com dados históricos — em andamento com time de dados

---

## 4. Métrica de sucesso

- **Métrica principal:** margem líquida média mensal por cartão ativo
  - Baseline: R$ 18,40/cartão/mês (média atual da carteira)
  - Meta: R$ 21,70/cartão/mês (+18%)
  - Janela de medição: 90 dias após início do rollout da fase 1

- **Métrica secundária:** % de cartões ativos com margem negativa
  - Baseline: 23%
  - Meta: < 12%

- **Guardrail:** taxa de aprovação no rollout não pode cair abaixo de 35% (risco de impacto em meta comercial)

---

## 5. Prazo para validação

90 dias após go-live da fase 1 do rollout (previsto para junho/2026)

---

## 6. Riscos e premissas

| Premissa / Risco | Probabilidade | Mitigação |
|---|---|---|
| Dados de custo operacional por cliente disponíveis no COST_ALLOC | Média | Mapear campos necessários com time de Finanças antes do PRD |
| Modelo de margem projetada pode ter viés por segmento sub-representado | Média | Validar com time de Risco e Atuarial antes de usar em produção |
| Área comercial pode resistir a restrição de oferta por segmento | Alta | Apresentar simulação de ganho de margem vs. perda de volume antes do rollout |
| Dados históricos de intercâmbio por cliente disponíveis no TXN_SYSTEM | Alta | Confirmar granularidade com Engenharia de Dados |

---

## 7. Impacto estimado

- **Nível:** Alto
- **Justificativa:** Produto estratégico do banco. Melhoria de margem impacta diretamente o resultado do produto no P&L.
- **Usuários/processos afetados:** ~480 mil clientes elegíveis na base atual, além de novos clientes na fila de ativação

---

## 8. Esforço percebido (PO)

- **Tamanho:** Grande
- **Observação:** Envolve múltiplos sistemas de origem (TXN, CORE, RISK, ACTU, COST_ALLOC), construção de motor de margem e mecanismo de rollout por segmento.

---

## 9. Fora do escopo

- Mudança nas regras de pricing do cartão (anuidade, juros, MDR) — escopo de Pricing
- Criação de novo produto de cartão — mantemos o Cartão Alfa
- Cobrança automática de seguro — sistema ACTU_SYSTEM já cobre isso

---

## 10. Origem inicial da informação

- **Sistemas prováveis:** TXN_SYSTEM (transações), CORE_BANK (cadastro/limite), RISK_ENGINE (PDD/score), ACTU_SYSTEM (seguro), COST_ALLOC (custos operacionais), FRAUD_SYS (fraude)
- **Área dona da informação:** Produto (Cartões), Risco de Crédito, Finanças, Atuarial
- **Tabelas/entidades conhecidas:** tb_transacao_cartao, tb_cliente_cartao, tb_limite_credito, tb_pdd_projecao — a confirmar com Engenharia de Dados
- **Pessoa ou time que pode validar:** Rafael Gomes (Risco de Crédito), Camila Neves (Finanças / Custos), Marcelo Souza (Atuarial)
- **Lacunas ainda existentes:** granularidade do custo operacional por cliente ainda não confirmada; modelo de projeção de intercâmbio por perfil de gasto a validar

---

## Gate — PM/PO + apoio consultivo do TL

- [x] Problema claro e embasado?
- [x] Hipótese testável com métrica e prazo?
- [x] Impacto e risco entendidos?
- [x] Origem inicial da informação minimamente identificada?
- [ ] Já existe base suficiente para evoluir para PRD?
- **Decisão:** Avançar para PRD — após confirmação da granularidade do COST_ALLOC com Camila Neves