# PRD - Exemplo simples de consulta e cálculo em contratos

## Em uma frase
- Disponibilizar uma tabela final simples de contratos com os principais campos necessários para consumo do negócio.

## Contexto do problema
- A área de acompanhamento comercial precisa consolidar uma visão simples de contratos para apoiar análises recorrentes de carteira.
- Hoje, sempre que essa informação é necessária, o time precisa pedir a consulta da base, confirmar quais campos devem ser considerados e reorganizar o resultado antes do uso.
- Isso aumenta o tempo de atendimento da demanda e abre espaço para diferentes interpretações sobre quais informações devem compor a saída final.
- Para evitar esse ruído, o negócio precisa registrar de forma objetiva qual tabela deve ser consultada, qual é a granularidade esperada, quais campos precisam ser trazidos e quais novos campos devem compor a entrega final.

## Objetivo do produto
- Atender a necessidade da área de acompanhamento comercial de receber uma visão padronizada por contrato, pronta para consumo.
- A entrega deve considerar uma tabela já conhecida pelo negócio como origem da informação.
- Para cada contrato, a saída deve trazer a chave do contrato, três campos de valor e mais dois campos calculados a partir de regras de negócio simples.
- O resultado esperado é uma tabela final clara, consistente e fácil de consumir em análises recorrentes.

## Para quem é essa entrega
- Área consumidora da informação
- PM/PA responsável pela demanda
- Squad que vai implementar a solução

## Escopo

### O que está dentro do escopo
| Item | Prioridade |
|---|---|
| Disponibilizar uma visão padronizada por contrato | Essencial |
| Garantir que a saída traga os principais campos necessários para análise | Essencial |
| Incluir os novos campos que o negócio precisa acompanhar | Essencial |
| Entregar uma estrutura estável para consumo recorrente | Essencial |
| Registrar de forma clara o que o negócio espera receber | Essencial |

### O que não está no escopo desta versão
| Item | Motivo |
|---|---|
| Contratos de outros produtos, como cartão e crédito pessoal | Esta entrega considera apenas a base de contratos definida para este cenário |
| Indicadores consolidados por gerente, regional ou produto | O foco desta versão é a visão no nível do contrato |
| Histórico mensal ou comparação entre períodos | Esta versão considera somente a entrega da visão atual da base consultada |

## Jornada esperada

### Situação atual
- O negócio sabe qual informação precisa receber, mas essa necessidade precisa ser organizada de forma clara antes de seguir para o time técnico.
- Para a apresentação, o objetivo é mostrar como essa clareza pode ser registrada em um documento simples.

### Situação desejada
- A necessidade fica descrita em linguagem de negócio.
- A origem da informação fica identificada.
- Os campos que precisam ser buscados ficam claros.
- A saída esperada fica definida antes da solução técnica.

## Origem da informação

| O que preciso | Tabela / sistema | Campos utilizados | Quem confirmou | Status | Observação |
|---|---|---|---|---|---|
| Informações do contrato | `tb_contratos_origem` | `chave_contrato`, `valor_1`, `valor_2`, `valor_3` | Time dono da informação | Validado para exemplo | Granularidade: 1 linha por contrato |

## Chaves que devem ser entregues
| Chave | Finalidade |
|---|---|
| `chave_contrato` | Identificar cada contrato na saída final |

## Variáveis ou artefatos que devem ser entregues
| Item | Leitura de negócio |
|---|---|
| `valor_1` | Primeiro valor que deve vir da origem |
| `valor_2` | Segundo valor que deve vir da origem |
| `valor_3` | Terceiro valor que deve vir da origem |
| `valor_calculado_1` | Novo campo calculado a partir dos valores de entrada |
| `valor_calculado_2` | Novo campo calculado a partir dos valores de entrada |
| `tb_contratos_saida` | Tabela final com todos os campos esperados pelo negócio |

## Regras de negócio
### Regra 1
A saída deve conter uma linha para cada contrato.

### Regra 2
A saída deve trazer a `chave_contrato` e os campos `valor_1`, `valor_2` e `valor_3`.

### Regra 3
O campo `valor_calculado_1` deve ser gerado a partir de uma regra simples de multiplicação entre os valores da origem.

`valor_calculado_1 = valor_1 * valor_2`

### Regra 4
O campo `valor_calculado_2` deve ser gerado a partir de uma regra simples de divisão entre os valores da origem.

`valor_calculado_2 = valor_3 / valor_2`

### Regra 5
Quando não for possível realizar o cálculo do segundo campo, esse campo não deve trazer valor.

### Regra 6
A tabela final deve conter:

- `chave_contrato`
- `valor_1`
- `valor_2`
- `valor_3`
- `valor_calculado_1`
- `valor_calculado_2`

## Critérios de aceite
| Situação | Resultado esperado |
|---|---|
| Contrato disponível na origem | Registro presente na saída com a chave e os campos esperados |
| Geração do primeiro novo campo | Campo entregue conforme a regra de negócio definida |
| Geração do segundo novo campo | Campo entregue conforme a regra de negócio definida |
| Exceção de cálculo | Campo sem valor quando a regra não puder ser aplicada |
| Estrutura final | Tabela final contém todos os campos previstos pelo negócio |

## Resultado esperado para o negócio
- Ter uma saída simples e padronizada para contratos.
- Deixar claro o que o negócio precisa receber.
- Facilitar o handoff para o time técnico sem exigir detalhamento de implementação no PRD.

## Métricas de sucesso
| Métrica | Meta inicial |
|---|---|
| Clareza da estrutura da saída | 100% dos campos identificados |
| Clareza da origem da informação | tabela, granularidade e campos definidos |
| Completude do exemplo | 1 exemplo completo de ponta a ponta |

## Dependências e alinhamentos
| Tema | Necessidade |
|---|---|
| Tabela de origem | Confirmar `tb_contratos_origem` |
| Time dono da informação | Confirmar qual time fornece essa base |
| Granularidade | Confirmar que a leitura é de 1 linha por contrato |
| Campos necessários | Confirmar `chave_contrato`, `valor_1`, `valor_2` e `valor_3` |

## Perguntas em aberto
| Questão | Responsável | Prazo | Impacto |
|---|---|---|---|
| Qual time é dono da tabela de origem neste exemplo | Negócio | Antes do handoff | Médio |

## Handoff
- [x] Origem da informação identificada
- [x] Granularidade da informação identificada
- [x] Campos necessários identificados
- [x] Campos obrigatórios identificados
- [x] Regras de negócio descritas
- [x] Critérios de aceite definidos
- [x] Saída final descrita
