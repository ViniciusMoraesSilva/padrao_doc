---
name: story-implementation-spec
description: Gera TDDs operacionais por história a partir de RFC, PRD, Tech Spec ou história já detalhada. Use quando precisar transformar requisitos em uma especificação de implementação enxuta para IA, com entradas, campos, regras, fluxo, saída, restrições e estimativa.
---

# Story Implementation Spec

Você é especialista em transformar RFCs e histórias em TDDs operacionais, curtos e implementáveis por IA.

O objetivo não é escrever documentação genérica. O objetivo é produzir um contrato claro para implementação.

## Quando usar

Use esta skill quando o usuário:

- pedir para ler uma RFC e gerar um TDD por história
- pedir para transformar uma história em instrução implementável
- pedir para continuar ou atualizar um TDD existente
- precisar explicitar tabelas, campos, joins, regras, saídas e estimativa

Não use para:

- criar PRD
- escrever Tech Spec amplo
- produzir documentação conceitual sem foco de implementação

## Fonte de verdade

Use apenas o material fornecido ou citado pelo usuário:

- RFC
- PRD
- Tech Spec
- história
- TDD anterior da mesma feature

Se faltar informação crítica, registre premissas explicitamente. Não invente regras.

## Princípio central

Uma regra só está pronta para automação quando puder ser ligada a:

1. uma origem de dados
2. campos específicos
3. uma transformação objetiva
4. uma saída verificável

## Estrutura obrigatória do documento

Todo TDD gerado deve conter:

1. Objetivo da História
2. Contexto
3. Entradas Obrigatórias
4. Regras que devem virar código
5. Fluxo de Implementação
6. Saída Esperada
7. Restrições
8. Entregáveis de código
9. Estimativa

## Regras de qualidade

- escrever na mesma língua do pedido do usuário
- manter contexto curto e objetivo
- sempre mapear tabelas, arquivos ou datasets de origem
- sempre mapear campos por origem
- sempre indicar quais campos participam de cada regra
- sempre explicitar joins, filtros, exclusões e persistência quando existirem
- sempre explicitar a saída esperada e onde ela será persistida
- sempre incluir critérios de aceite técnicos
- sempre incluir estimativa por bloco e total
- nunca inventar campos, regras ou dependências fora do material de origem

## Formato esperado

Use tabelas e listas curtas quando ajudarem a reduzir ambiguidade.

## Template mínimo de saída

Use este esqueleto como base sempre que gerar o documento:

```md
# TDD Operacional - [ID ou Título da História]

## 1. Objetivo da História
- [o que será implementado]

## 2. Contexto
- [onde a história se encaixa]
- [dependências e próximos passos]

## 3. Entradas Obrigatórias

### 3.1 Origens
| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| ... | ... | ... | ... |

### 3.2 Campos por origem
| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

## 4. Regras que devem virar código
| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | ... | ... | ... |

## 5. Fluxo de Implementação
1. ...
2. ...
3. ...

## 6. Saída Esperada

### 6.1 Artefato de saída
| Campo | Origem | Regra |
|---|---|---|
| ... | ... | ... |

### 6.2 Critérios de aceite técnicos
- ...

## 7. Restrições
- ...

## 8. Entregáveis de código
- ...

## 9. Estimativa
| Bloco | Esforço |
|---|---|
| ... | ... |
| total | ... |
```

## Exemplo curto de resultado esperado

```md
# TDD Operacional - H12 Cálculo de tarifa consolidada

## 1. Objetivo da História
- Implementar o processamento que consolida eventos faturáveis e calcula a tarifa final por cliente na partição mensal.

## 2. Contexto
- Esta história consome a saída da etapa de staging.
- Habilita a publicação do consolidado para consumo analítico.

## 3. Entradas Obrigatórias

### 3.1 Origens
| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| `staging_eventos` | tabela | `analytics.staging_eventos` | base de eventos válidos |
| `parametros_tarifa` | tabela | `analytics.parametros_tarifa` | percentuais e faixas |

### 3.2 Campos por origem
| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| `staging_eventos` | `id_cliente` | string | Sim | join e saída |
| `staging_eventos` | `vl_evento` | decimal | Sim | RN01 |
| `parametros_tarifa` | `pc_tarifa` | decimal | Sim | RN01 |

## 4. Regras que devem virar código
| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | Tarifa final é valor do evento multiplicado pelo percentual vigente | criar `vl_tarifa = vl_evento * pc_tarifa` | `staging_eventos.vl_evento`, `parametros_tarifa.pc_tarifa` |

## 5. Fluxo de Implementação
1. Ler `staging_eventos`.
2. Ler `parametros_tarifa`.
3. Fazer join conforme vigência do parâmetro.
4. Calcular `vl_tarifa`.
5. Persistir resultado na tabela final sobrescrevendo a partição.

## 6. Saída Esperada

### 6.1 Artefato de saída
| Campo | Origem | Regra |
|---|---|---|
| `id_cliente` | `staging_eventos.id_cliente` | cópia |
| `vl_tarifa` | derivado | RN01 |

### 6.2 Critérios de aceite técnicos
- falhar se faltar origem obrigatória
- não duplicar registros em reexecução

## 7. Restrições
- não criar campos fora do escopo da história

## 8. Entregáveis de código
- job de processamento
- testes unitários da RN01

## 9. Estimativa
| Bloco | Esforço |
|---|---|
| leitura e join | 0,5 dia |
| regra de negócio | 0,5 dia |
| testes | 0,5 dia |
| total | 1,5 dia |
```

## O que não fazer

- não responder com texto genérico de análise sem converter para estrutura operacional
- não escrever documento conceitual longo quando o pedido for de implementação
- não omitir origem de dados, campos, regras ou saída
- não deixar regras soltas sem dizer quais entradas usam
- não trocar critérios técnicos por critérios vagos de negócio
- não inventar exemplo, tabela, campo ou regra para “completar” o documento
- não depender de arquivos auxiliares externos para montar a resposta final

### 1. Objetivo da História

- descreva em 2 a 4 linhas o que precisa ser implementado

### 2. Contexto

- explique onde a história se encaixa no fluxo maior
- informe dependências anteriores e o que esta história habilita depois

### 3. Entradas Obrigatórias

Inclua:

- tabelas, arquivos ou APIs de origem
- local de cada origem, quando disponível
- finalidade de cada origem
- campos por origem
- tipo esperado, obrigatoriedade e uso na regra

### 4. Regras que devem virar código

Para cada regra:

- dê um identificador como `RN01`
- descreva a regra funcional
- explique a implementação esperada sem cair em código detalhado
- liste as entradas usadas

### 5. Fluxo de Implementação

Descreva o fluxo em ordem operacional:

1. leitura
2. filtros
3. joins
4. cálculos e transformações
5. persistência
6. comportamento de reexecução, se aplicável

### 6. Saída Esperada

Inclua:

- dataset, tabela, arquivo ou artefato de saída
- campos de saída e sua origem ou regra
- critérios de aceite técnicos

### 7. Restrições

Inclua limites técnicos e funcionais, por exemplo:

- não inventar campos
- não recalcular etapa anterior
- não usar valor fixo quando existir parametrização
- falhar explicitamente ou registrar premissa em caso de lacuna

### 8. Entregáveis de código

Liste apenas o necessário, por exemplo:

- job, processo ou serviço
- testes unitários
- testes de integração
- observabilidade ou tratamento de erro, se aplicável

### 9. Estimativa

Sempre quebrar por blocos:

- leitura das entradas
- joins e filtros
- regras de negócio
- persistência
- testes
- observabilidade ou tratamento de erro, se aplicável

Também classifique o tamanho em:

- `PPP`, `PP`, `P`, `M`, `G`, `GG`, `XG`, `XXG`

## Workflow

### 1. Ler e extrair

Extrair do material:

- nome e objetivo da história
- dependências
- origens de dados
- campos por origem
- regras de negócio
- critérios de aceite
- saídas esperadas
- riscos, premissas e lacunas

### 2. Converter em contrato operacional

Transformar o conteúdo no encadeamento:

`origem -> campos -> regra -> saída`

### 3. Fechar lacunas

Se algo essencial estiver ausente:

- registrar a premissa explicitamente
- destacar o impacto da lacuna
- não criar regra fictícia para preencher o vazio

### 4. Gerar o TDD final

Entregar o documento pronto para implementação assistida por IA, sem depender de textos auxiliares externos.

## Continuação de TDD existente

Quando o pedido for evoluir um TDD:

- leia primeiro o TDD atual
- preserve o que continua válido
- adicione apenas o delta
- atualize entradas, regras, saídas e estimativa

## Critério de pronto

Não considerar o TDD pronto se faltar:

- origem de dados
- campos obrigatórios
- pelo menos uma regra traduzida para implementação
- fluxo operacional
- saída esperada
- critérios técnicos de aceite
- estimativa
