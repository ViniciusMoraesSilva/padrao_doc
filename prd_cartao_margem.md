# PRD — Cálculo de Margem de Rentabilidade e Rollout do Cartão Alfa

**Hipótese de origem:** hipotese_cartao_margem.md
**Squad:** Squad Cartões · **PM/PO:** Ana Beatriz Torres · **TL:** Bruno Ferreira
**Data:** 21/04/2026 · **Status:** Em review

---

## 1. Objetivo e contexto

Hoje o banco não sabe se está ganhando ou perdendo dinheiro com cada cliente que tem o Cartão Alfa. A decisão de oferecer o cartão é baseada só em score de crédito e renda mínima, sem considerar o quanto o cliente usa o cartão, quanto paga de juros ou qual é o risco de não pagar.

O objetivo desse projeto é calcular a margem de cada cliente — ou seja, quanto o banco ganha com aquele cliente depois de descontar todos os custos — e usar esse número para decidir para quem oferecer o cartão primeiro. Quem tiver margem maior entra na primeira onda do rollout. Quem tiver margem negativa não entra nas primeiras fases.

---

## 2. Quem é afetado

- **Gestores de Produto (Cartões):** passam a ter visibilidade de margem por grupo de cliente antes de acionar uma campanha
- **Time Comercial:** recebe uma lista de clientes organizada por quem tem maior margem, pronta para usar no rollout
- **Time de Risco:** valida se as premissas do cálculo estão corretas e acompanha mês a mês

---

## 3. Escopo

| O que será feito | Prioridade | Por quê |
|---|---|---|
| Calcular a margem projetada de cada cliente | Indispensável | É a base de tudo |
| Classificar os clientes em grupos (Alta Margem / Margem Média / Margem Baixa) | Indispensável | É o critério do rollout |
| Gerar lista de clientes por grupo para a área comercial usar no rollout | Indispensável | Para a campanha funcionar |
| Guardar o histórico dos cálculos para consulta futura | Indispensável | Auditoria e acompanhamento mês a mês |
| Permitir que Finanças ajuste os valores fixos do cálculo sem depender de TI | Importante | Finanças precisa ajustar o custo fixo mensalmente |
| Calcular margem de clientes que nunca tiveram cartão | Não será feito agora | Esses clientes não têm histórico de gasto — escopo futuro |

---

## 4. Como o processo vai funcionar

1. No começo de cada mês, o time de TI roda o cálculo para todos os clientes da base
2. O sistema busca as informações de cada cliente nas tabelas do catálogo de dados (Blue)
3. Com essas informações, calcula quanto o banco vai ganhar com aquele cliente (receitas) e quanto vai custar atendê-lo (custos)
4. Subtrai os custos das receitas e chega na margem líquida do cliente
5. Com base na margem, classifica o cliente em um dos três grupos: **Alta Margem**, **Margem Média** ou **Margem Baixa**
6. Gera uma lista com os clientes de Alta Margem e Margem Média para o time comercial usar na campanha de rollout
7. O time comercial é avisado quando a lista está pronta

---

## 5. Critérios de aceite de negócio

| Situação | O que deve acontecer |
|---|---|
| O cliente tem histórico de compras no cartão | O cálculo usa a média mensal dos últimos 12 meses de receita de intercâmbio |
| O cliente ficou com saldo em aberto na fatura | O cálculo considera a receita de juros sobre esse saldo |
| O cliente tem score de crédito de 700 ou mais | O custo de inadimplência é 2,1% do limite aprovado |
| O cliente tem score entre 620 e 699 | O custo de inadimplência é 4,8% do limite aprovado |
| O cliente tem score entre 500 e 619 | O custo de inadimplência é 9,3% do limite aprovado |
| O cliente tem score abaixo de 500 | O cliente não entra no cálculo e não recebe oferta do cartão |
| A margem calculada for R$ 60,00 ou mais | O cliente é classificado como **Alta Margem** |
| A margem for entre R$ 20,00 e R$ 59,99 | O cliente é classificado como **Margem Média** |
| A margem for menos de R$ 20,00 | O cliente é classificado como **Margem Baixa** e não entra nas fases 1 e 2 do rollout |
| Alguma das informações necessárias não estiver disponível | O cálculo não pode ser feito com dado incompleto — o time de TI deve ser avisado |

---

## 6. Métricas de sucesso

- **Primária:** margem líquida média mensal por cartão ativo — baseline R$ 18,40 → meta R$ 21,70
- **Secundária:** % de cartões com margem negativa — baseline 23% → meta < 12%
- **Guardrail:** taxa de aprovação no rollout ≥ 35%
- **Janela de medição:** 90 dias após go-live da fase 1

---

## 7. De onde vêm as informações

Todas as informações estão disponíveis no catálogo de dados da empresa (Blue). O PO identificou abaixo as tabelas e os campos necessários para cada parte do cálculo, com o responsável por cada uma delas.

| Informação necessária | Tabela no catálogo | Campos que serão usados | Responsável | Observações |
|---|---|---|---|---|
| Histórico de compras e a taxa gerada por cada compra (intercâmbio) | `transacoes_cartao` | `nr_cpf`, `vl_mdr`, `cd_tipo_transacao`, `dt_transacao` | Carla Dias — Engenharia de Dados | Usar apenas registros onde o tipo é "COMPRA". Precisamos dos últimos 12 meses. |
| Saldo em aberto do cliente na fatura do cartão | `fatura_cartao` | `nr_cpf`, `vl_saldo_devedor`, `dt_referencia` | Core Banking | Precisamos da média dos últimos 12 meses. Confirmar se a tabela tem esse histórico. |
| Limite de crédito aprovado para o cliente | `limite_credito` | `nr_cpf`, `vl_limite_aprovado` | Core Banking | Usar o limite aprovado atual. |
| Score de crédito do cliente | `cliente_cartao` | `nr_cpf`, `nr_score_credito` | Core Banking | Score do momento em que o cálculo for rodado. |
| Percentual de inadimplência esperado por faixa de score | `pdd_faixas_score` | `score_min`, `score_max`, `pc_pdd` | Rafael Gomes — Risco de Crédito | Essa tabela é atualizada pelo time de Risco. Sempre usar a versão mais recente. |

---

## 8. Regras de negócio

### Receitas que o banco tem com o cliente

**RN01 — Receita de intercâmbio**
Toda vez que o cliente faz uma compra no cartão, o banco recebe uma taxa chamada MDR. A receita de intercâmbio é a média mensal dessa taxa ao longo dos últimos 12 meses.
- Tabela: `transacoes_cartao`
- Campos: `vl_mdr`, `cd_tipo_transacao`, `dt_transacao`
- Filtro: usar apenas transações do tipo "COMPRA"
- Cálculo: total de MDR por mês → média dos 12 meses
- Validação: Carla Dias (Engenharia de Dados)

**RN02 — Receita de juros do rotativo**
Quando o cliente não paga a fatura inteira, o banco cobra juros sobre o saldo em aberto. A taxa do produto é de 12,5% ao mês sobre o saldo médio em aberto nos últimos 12 meses. Se o cliente nunca ficou com saldo em aberto, essa receita é zero.
- Tabela: `fatura_cartao`
- Campo: `vl_saldo_devedor`
- Cálculo: média do saldo dos últimos 12 meses × 12,5%
- Validação: Rafael Gomes (Risco de Crédito)

---

### Custos que o banco tem com o cliente

**RN03 — Custo de inadimplência**
O banco reserva uma parte do limite de crédito para cobrir o risco de o cliente não pagar. Esse percentual varia conforme o score do cliente:

| Score do cliente | Percentual reservado |
|---|---|
| 700 ou mais | 2,1% do limite aprovado |
| Entre 620 e 699 | 4,8% do limite aprovado |
| Entre 500 e 619 | 9,3% do limite aprovado |
| Abaixo de 500 | Cliente não entra no cálculo |

- Tabelas: `cliente_cartao` (score), `limite_credito` (limite), `pdd_faixas_score` (percentuais)
- Validação: Rafael Gomes (Risco de Crédito)

**RN04 — Custo operacional**
O banco tem um custo fixo mensal por cartão ativo que cobre emissão, manutenção e atendimento. Esse valor é de **R$ 9,20 por cliente por mês**, definido pela área de Finanças. Pode ser ajustado sem necessidade de mudança no sistema.
- Valor: R$ 9,20 fixo por cliente
- Validação: Camila Neves (Finanças)

---

### Cálculo final

**RN05 — Margem líquida e classificação do cliente**

> Margem = (Receita de intercâmbio + Receita de juros) − (Custo de inadimplência + Custo operacional)

Com base na margem calculada, o cliente é classificado em:

| Margem calculada | Grupo | Entra no rollout? |
|---|---|---|
| R$ 60,00 ou mais | **Alta Margem** | Sim — Fase 1 |
| Entre R$ 20,00 e R$ 59,99 | **Margem Média** | Sim — Fase 2 |
| Menos de R$ 20,00 | **Margem Baixa** | Não entra nas fases 1 e 2 |

- Validação das faixas: Ana Torres (PO) + Camila Neves (Finanças)

---

## 9. Como é hoje

Hoje o banco decide se vai oferecer o Cartão Alfa para um cliente com base em duas perguntas:
1. O score de crédito é 500 ou mais?
2. A renda declarada é R$ 1.500 ou mais?

Se as duas respostas forem sim, o cliente recebe a oferta. Não existe nenhum cálculo de rentabilidade envolvido nessa decisão.

O que muda com esse projeto: além dessas duas condições, o sistema vai calcular a margem do cliente e usar esse número para definir em qual fase do rollout ele vai entrar.

**Pontos ainda não confirmados:**
- A tabela `fatura_cartao` guarda o histórico dos últimos 12 meses ou só a fatura mais recente? — confirmar com Core Banking até 25/04/2026
- O campo `vl_mdr` está disponível para todos os clientes na tabela `transacoes_cartao`? — confirmar com Carla Dias até 25/04/2026

---

## 10. Glossário

| Termo | O que significa neste contexto |
|---|---|
| Margem líquida | Quanto o banco ganha com um cliente depois de pagar todos os custos |
| Intercâmbio (MDR) | Taxa que o banco recebe de quem vendeu quando o cliente faz uma compra no cartão |
| Rotativo | Quando o cliente não paga a fatura inteira e fica com saldo em aberto pagando juros |
| PDD | Reserva financeira para cobrir o risco de inadimplência |
| Score de crédito | Nota de 0 a 1000 que indica o risco de o cliente não pagar |
| Rollout | Lançamento gradual do produto, começando pelos clientes mais rentáveis |
| Catálogo de dados (Blue) | Repositório central onde ficam todas as tabelas de dados da empresa |

---

## 11. Dependências e prazos

- Validação das regras de inadimplência (RN03) com Rafael Gomes (Risco de Crédito) — até 25/04/2026
- Confirmação do histórico disponível na tabela `fatura_cartao` com Core Banking — até 25/04/2026
- Confirmação da disponibilidade do campo `vl_mdr` com Carla Dias (Engenharia de Dados) — até 25/04/2026
- A lista de rollout da fase 1 precisa estar pronta até **26/05/2026** para a campanha comercial de 02/06/2026

---

## 12. O que não é responsabilidade do PO neste documento

Este PRD descreve **o quê** precisa ser calculado e **com quais informações**. Como o time técnico vai processar esses dados é responsabilidade do Tech Lead e do time, e será descrito na Tech Spec.

---

## 13. Perguntas em aberto

- [ ] A tabela `fatura_cartao` tem histórico dos últimos 12 meses ou só a fatura mais recente? — responsável: Core Banking — prazo: 25/04/2026
- [ ] O campo `vl_mdr` está disponível para todos os clientes na tabela `transacoes_cartao`? — responsável: Carla Dias — prazo: 25/04/2026

---

## Gate de handoff para o Tech Lead

- [x] Objetivo claro e em linguagem de negócio?
- [x] Critérios de aceite cobrem os casos principais e as exceções?
- [x] Fluxo do processo descrito sem termos técnicos?
- [x] Tabelas e campos no catálogo identificados com o responsável por cada um?
- [x] Regras de negócio descritas com o dado necessário e quem valida?
- [ ] Perguntas em aberto resolvidas? (2 pendentes — ver seção 13)
- [x] O Tech Lead consegue começar a solução sem precisar descobrir a regra no lugar do PO?
