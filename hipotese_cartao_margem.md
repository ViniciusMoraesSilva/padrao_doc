# Documento de Hipótese — Cálculo de Margem e Rollout do Cartão Alfa

**Autor:** Ana Beatriz Torres (PO)
**Data:** 14/04/2026
**Squad:** Squad Cartões
**Status:** Aprovado

---

## 1. Problema observado

O Cartão Alfa é ofertado hoje para todos os clientes elegíveis com base apenas em score de crédito e renda mínima. O banco não tem visibilidade de rentabilidade individual por cliente.

Com isso, cartões são emitidos para segmentos com alta inadimplência e baixo volume de gastos, gerando margem líquida negativa para uma parcela relevante da carteira.

---

## 2. Hipótese de solução

Se calcularmos a margem de rentabilidade projetada por cliente (receitas estimadas menos custos estimados) e usarmos esse cálculo como critério de priorização no rollout, acreditamos que a margem média do produto vai aumentar em pelo menos 18%, porque o banco passará a ativar primeiro os clientes com maior potencial de retorno e menor risco de custo.

---

## 3. Evidências e contexto

- [x] Análise interna (Dez/2025): 23% da carteira atual opera com margem líquida mensal negativa
- [x] Relatório de Risco (Jan/2026): clientes com score entre 500 e 620 concentram 61% do PDD total do produto
- [x] Benchmark setorial: bancos com motor de rentabilidade por produto reportam margem 22% superior à média

---

## 4. Métrica de sucesso

- **Métrica principal:** margem líquida média mensal por cartão ativo
  - Baseline: R$ 18,40/cartão/mês
  - Meta: R$ 21,70/cartão/mês (+18%)
  - Janela de medição: 90 dias após início do rollout fase 1

- **Métrica secundária:** % de cartões com margem negativa
  - Baseline: 23%
  - Meta: < 12%

- **Guardrail:** taxa de aprovação no rollout não pode cair abaixo de 35%

---

## 5. Prazo para validação

90 dias após go-live da fase 1 do rollout (previsto para junho/2026).

---

## 6. Riscos e premissas

| Premissa / Risco | Probabilidade | Mitigação |
|---|---|---|
| O histórico de transações por cliente está disponível no catálogo de dados com granularidade suficiente para calcular a média dos últimos 12 meses | Alta | Confirmar com Carla Dias (Engenharia de Dados) antes do PRD |
| As informações de saldo devedor, limite aprovado e score estão acessíveis para o time de TI | Média | Validar disponibilidade com Core Banking antes do PRD |
| Área comercial pode resistir à restrição de oferta por segmento | Alta | Apresentar simulação de ganho de margem antes do rollout |

---

## 7. Impacto estimado

- **Nível:** Alto
- **Justificativa:** Produto estratégico. Melhoria de margem impacta diretamente o P&L do banco.
- **Usuários/processos afetados:** ~480 mil clientes elegíveis na base atual

---

## 8. Esforço percebido (PO)

- **Tamanho:** Médio
- **Observação:** Envolve dados de 3 tabelas do catálogo e 5 regras de negócio. O time de TI vai definir como processar.

---

## 9. Fora do escopo

- Mudança nas regras de pricing do cartão (anuidade, juros, MDR)
- Exposição de margem via API para sistemas externos
- Cálculo de margem para novos clientes sem histórico (escopo futuro)

---

## 10. Origem inicial da informação

- **Tabelas conhecidas no catálogo de dados (Blue):**
  - `transacoes_cartao` — histórico de compras e a taxa de intercâmbio gerada por cada transação
  - `fatura_cartao` — saldo em aberto do cliente na fatura do cartão
  - `limite_credito` — limite de crédito aprovado por cliente
  - `cliente_cartao` — score de crédito do cliente
  - `pdd_faixas_score` — percentual de inadimplência esperado por faixa de score

- **Área dona da informação:** Engenharia de Dados, Risco de Crédito, Core Banking
- **Pessoa ou time que pode validar:** Rafael Gomes (Risco de Crédito), Camila Neves (Finanças), Carla Dias (Engenharia de Dados)
- **Lacunas ainda existentes:** confirmar com Carla Dias se a tabela `transacoes_cartao` tem histórico dos últimos 12 meses disponível por cliente

---

## Gate — PM/PO + apoio consultivo do TL

- [x] Problema claro e embasado?
- [x] Hipótese testável com métrica e prazo?
- [x] Impacto e risco entendidos?
- [x] Origem inicial da informação minimamente identificada?
- [x] Já existe base suficiente para evoluir para PRD?
- **Decisão:** Avançar para PRD
