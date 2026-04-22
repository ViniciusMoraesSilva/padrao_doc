# PRD — [Nome da iniciativa]

**PM/PO:** [Nome] · **Data:** [dd/mm/aaaa] · **Área:** [Nome da área]

---

## Em uma frase

[Descreva em uma frase o que será entregue para o negócio.]

---

## Contexto do problema

[Explique o problema atual em linguagem de negócio. Foque em dor real, impacto no dia a
dia e por que o cenário atual não atende.]

Na prática, isso gera:

1. [Impacto 1]
2. [Impacto 2]
3. [Impacto 3]

[Feche com a necessidade de negócio que a iniciativa resolve.]

---

## Objetivo do produto

[Descreva o objetivo principal da entrega.]

Esta iniciativa deve entregar:

- [Item 1]
- [Item 2]
- [Item 3]

[Se necessário, deixe explícito o que esta iniciativa não pretende resolver agora.]

---

## Para quem é essa entrega

**[Persona ou sistema consumidor 1]**
> "[Frase em primeira pessoa descrevendo a dor ou necessidade.]"

O que precisa: [descrição objetiva]

**[Persona ou sistema consumidor 2]**
> "[Frase em primeira pessoa]"

O que precisa: [descrição objetiva]

**[Persona ou sistema consumidor 3]**
> "[Frase em primeira pessoa]"

O que precisa: [descrição objetiva]

---

## Escopo

### O que está dentro do escopo

| Item | Prioridade |
|---|---|
| [Entrega dentro do escopo] | Essencial |
| [Entrega dentro do escopo] | Essencial |
| [Entrega dentro do escopo] | Importante |

### O que não está no escopo desta versão

| Item | Motivo |
|---|---|
| [Fora do escopo] | [Justificativa] |
| [Fora do escopo] | [Justificativa] |
| [Fora do escopo] | [Justificativa] |

---

## Jornada esperada

### Situação atual

[Descreva como a área trabalha hoje ou como o dado é consumido hoje.]

### Situação desejada

[Descreva como ficará depois da entrega.]

- [Resultado esperado 1]
- [Resultado esperado 2]
- [Resultado esperado 3]

[Feche com o ganho esperado.]

---

## Origem da informação

[Descreva de forma simples qual é a base de entrada da iniciativa.]

Premissas de negócio:

- [Premissa 1]
- [Premissa 2]
- [Premissa 3]

### Fonte de dados de entrada

| O que preciso | Tabela / sistema | Campos utilizados | Quem confirmou | Status | Observação |
|---|---|---|---|---|---|
| [Informação de negócio] | `[tabela_ou_sistema]` | `[campo_1, campo_2]` | [Nome] | Validado em [data] | [Observação] |
| [Informação de negócio] | `[tabela_ou_sistema]` | `[campo_1]` | [Nome] | Pendente até [data] | [Observação] |
| [Informação de negócio] | `[tabela_ou_sistema]` | `[campo_1]` | [Nome] | Com restrição | [Observação] |

### Regra de preenchimento desta seção

Quem escreve o PRD deve informar:

- qual tabela será usada
- quais campos da origem serão consumidos
- quem validou a existência e o significado desses campos
- se a informação está validada, pendente ou com restrição conhecida

Se algum campo necessário ainda não estiver validado, isso deve aparecer explicitamente no
documento antes do handoff para o time técnico.

---

## Chaves que devem ser entregues

Cada registro da saída deve ser identificado pelas seguintes chaves:

| Chave | Finalidade |
|---|---|
| `[chave_1]` | [Descrição] |
| `[chave_2]` | [Descrição] |
| `[chave_3]` | [Descrição] |
| `[chave_4]` | [Descrição] |

---

## Variáveis de valor que o sistema deve receber

O sistema consumidor deve receber as seguintes variáveis de valor:

| Variável | Leitura de negócio |
|---|---|
| `[variavel_1]` | [Descrição de negócio] |
| `[variavel_2]` | [Descrição de negócio] |
| `[variavel_3]` | [Descrição de negócio] |
| `[variavel_4]` | [Descrição de negócio] |

---

## Regras de negócio

### Regra 1

[Descreva a regra funcional principal.]

### Regra 2

[Descreva outra regra funcional importante.]

### Regra 3

[Descreva outra regra funcional importante.]

### Regra 4 — Cálculo de `[campo_calculado_1]`

`[campo_calculado_1] = [expressão de cálculo]`

[Explique em linguagem de negócio o racional da regra.]

### Regra 5 — Cálculo de `[campo_calculado_2]`

`[campo_calculado_2] = [expressão de cálculo]`

### Regra 6 — Cálculo de `[campo_calculado_3]`

`[campo_calculado_3] = [expressão de cálculo]`

### Regra 7

[Explique exceções relevantes, como nulo, zero, vencido ou informação ausente.]

Observação para o autor:
- sempre que um campo da saída for resultado de cálculo, a regra deve estar explícita
- a área de negócio é responsável por aprovar a lógica descrita aqui antes do handoff

---

## Critérios de aceite

| Situação | Resultado esperado |
|---|---|
| [Caso principal] | [Resultado esperado] |
| [Registro válido] | [Resultado esperado] |
| [Campo calculado 1] | [Resultado esperado] |
| [Campo calculado 2] | [Resultado esperado] |
| [Exceção] | [Resultado esperado] |
| [Novo ciclo de execução] | [Resultado esperado] |

---

## Resultado esperado para o negócio

[Descreva o resultado esperado ao final da entrega.]

O benefício esperado é:

- [Benefício 1]
- [Benefício 2]
- [Benefício 3]
- [Benefício 4]

---

## Métricas de sucesso

| Métrica | Meta inicial |
|---|---|
| [Métrica] | [Meta] |
| [Métrica] | [Meta] |
| [Métrica] | [Meta] |
| [Métrica] | [Meta] |

---

## Dependências e alinhamentos

| Tema | Necessidade |
|---|---|
| [Dependência] | [Descrição] |
| [Dependência] | [Descrição] |
| [Dependência] | [Descrição] |
| [Dependência] | [Descrição] |

---

## Perguntas em aberto

| Questão | Responsável | Prazo | Impacto |
|---|---|---|---|
| [Pergunta em aberto] | [Nome] | [Data] | Alto |
| [Pergunta em aberto] | [Nome] | [Data] | Médio |
| [Pergunta em aberto] | [Nome] | [Data] | Baixo |

---

## Handoff

- [ ] Problema de negócio está claro
- [ ] Objetivo da entrega está claro
- [ ] Entrada está definida em linguagem de negócio
- [ ] Tabela e campos de origem foram informados
- [ ] Status de validação das informações foi registrado
- [ ] Chaves do registro estão definidas
- [ ] Variáveis de valor estão nomeadas
- [ ] Regras de cálculo dos campos derivados estão explícitas
- [ ] Critérios de aceite cobrem caminho principal e exceções
- [ ] Pendências de negócio foram resolvidas
