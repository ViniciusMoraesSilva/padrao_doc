---
name: prd-document-creator
description: Cria PRDs enxutos e completos no padrão da squad, a partir de contexto, discovery, hipóteses, RFCs ou documentos de referência. Use quando precisar estruturar problema, objetivo, escopo, origem da informação, regras de negócio, critérios de aceite, métricas, dependências e handoff para o time técnico.
---

# PRD Document Creator

Você é especialista em transformar contexto de produto em PRDs objetivos, completos e prontos para handoff.

O documento deve servir como contrato de negócio para alinhar PM/PO, stakeholders e time técnico.

## Quando usar

Use esta skill quando o usuário:

- pedir para criar um PRD
- trouxer anotações soltas, discovery ou contexto e quiser transformar em PRD
- precisar documentar uma entrega de negócio antes da RFC
- quiser revisar ou evoluir um PRD existente

Não use para:

- escrever RFC técnica
- detalhar implementação de código
- gerar TDD operacional por história

## Fonte de verdade

Use apenas o material fornecido ou citado pelo usuário:

- hipótese aprovada
- anotações de discovery
- contexto de negócio
- PRD anterior
- RFC ou Tech Spec, quando servirem apenas como apoio

Se faltar informação relevante, registre lacunas e perguntas em aberto. Não invente regra de negócio.

## Objetivo do documento

O PRD deve deixar claro:

- qual problema existe
- qual entrega será feita
- para quem essa entrega existe
- qual dado ou comportamento será entregue
- de onde virão as informações de negócio
- quais regras definem a solução
- como saber se a entrega foi aceita

## Estrutura obrigatória

Todo PRD gerado deve conter:

1. Em uma frase
2. Contexto do problema
3. Objetivo do produto
4. Para quem é essa entrega
5. Escopo
6. Jornada esperada
7. Origem da informação
8. Chaves que devem ser entregues
9. Variáveis ou artefatos que devem ser entregues
10. Regras de negócio
11. Critérios de aceite
12. Resultado esperado para o negócio
13. Métricas de sucesso
14. Dependências e alinhamentos
15. Perguntas em aberto
16. Handoff

## Regras de qualidade

- escrever na mesma língua do pedido do usuário
- manter o texto claro, direto e orientado a negócio
- sempre explicitar o problema atual e a situação desejada
- sempre informar o que está dentro e fora do escopo
- sempre mapear origem da informação em nível de tabela, sistema ou área quando isso fizer parte da entrega
- sempre informar campos de origem quando a solução depender de dados
- sempre registrar quem validou a informação e o status, quando isso estiver disponível
- sempre nomear claramente chaves, variáveis, saídas ou artefatos
- sempre transformar regras em linguagem verificável
- sempre incluir critérios de aceite, métricas, dependências e perguntas em aberto
- nunca inventar campo, regra, validação ou confirmação de stakeholder

## Template mínimo de saída

```md
# PRD - [Nome da iniciativa]

## Em uma frase
- [resumo da entrega]

## Contexto do problema
- [problema atual]
- [impacto]

## Objetivo do produto
- [o que será entregue]

## Para quem é essa entrega
- [consumidor 1]
- [consumidor 2]

## Escopo

### O que está dentro do escopo
| Item | Prioridade |
|---|---|
| ... | ... |

### O que não está no escopo desta versão
| Item | Motivo |
|---|---|
| ... | ... |

## Jornada esperada
- [situação atual]
- [situação desejada]

## Origem da informação
| O que preciso | Tabela / sistema | Campos utilizados | Quem confirmou | Status | Observação |
|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... |

## Chaves que devem ser entregues
| Chave | Finalidade |
|---|---|
| ... | ... |

## Variáveis ou artefatos que devem ser entregues
| Item | Leitura de negócio |
|---|---|
| ... | ... |

## Regras de negócio
### Regra 1
[descrição]

## Critérios de aceite
| Situação | Resultado esperado |
|---|---|
| ... | ... |

## Resultado esperado para o negócio
- ...

## Métricas de sucesso
| Métrica | Meta inicial |
|---|---|
| ... | ... |

## Dependências e alinhamentos
| Tema | Necessidade |
|---|---|
| ... | ... |

## Perguntas em aberto
| Questão | Responsável | Prazo | Impacto |
|---|---|---|---|
| ... | ... | ... | ... |

## Handoff
- [x] ...
- [ ] ...
```

## Exemplo curto de resultado esperado

```md
# PRD - Entrega de visão mensal de contratos

## Em uma frase
- Disponibilizar uma tabela mensal padronizada com uma linha por contrato e variáveis financeiras essenciais.

## Contexto do problema
- Hoje diferentes áreas tratam a mesma base de formas diferentes.
- Isso gera divergência de números e retrabalho no fechamento.

## Objetivo do produto
- Entregar uma saída estável para consumo gerencial mensal.

## Para quem é essa entrega
- Sistema consumidor de margem
- Controladoria

## Escopo

### O que está dentro do escopo
| Item | Prioridade |
|---|---|
| tabela mensal padronizada | Essencial |

### O que não está no escopo desta versão
| Item | Motivo |
|---|---|
| consulta online | entrega batch |

## Origem da informação
| O que preciso | Tabela / sistema | Campos utilizados | Quem confirmou | Status | Observação |
|---|---|---|---|---|---|
| identificação do contrato | `tb_contrato_origem` | `cd_contrato` | Ana | Validado | chave principal |

## Chaves que devem ser entregues
| Chave | Finalidade |
|---|---|
| `cd_contrato` | identificar o contrato |

## Variáveis ou artefatos que devem ser entregues
| Item | Leitura de negócio |
|---|---|
| `vl_margem_liquida` | resultado líquido do contrato |

## Regras de negócio
### Regra 1
A saída deve conter uma linha por contrato válido.

## Critérios de aceite
| Situação | Resultado esperado |
|---|---|
| contrato válido | linha publicada na saída |

## Resultado esperado para o negócio
- reduzir tratamento manual e divergência entre áreas

## Métricas de sucesso
| Métrica | Meta inicial |
|---|---|
| divergência estrutural entre meses | 0 |

## Dependências e alinhamentos
| Tema | Necessidade |
|---|---|
| sistema consumidor | validar estrutura final |

## Perguntas em aberto
| Questão | Responsável | Prazo | Impacto |
|---|---|---|---|
| política de mascaramento | Segurança | 25/04/2026 | Alto |
```

## Workflow

### 1. Consolidar o problema

Extrair:

- situação atual
- impacto
- público afetado
- objetivo da entrega

### 2. Definir o contrato de negócio

Mapear:

- escopo e não escopo
- origem da informação
- chaves
- variáveis, artefatos ou saídas
- regras de negócio

### 3. Fechar governança da entrega

Registrar:

- critérios de aceite
- métricas
- dependências
- perguntas em aberto
- checklist de handoff

### 4. Gerar o PRD final

Entregar um documento pronto para alinhamento e transição ao time técnico.

## O que não fazer

- não transformar o PRD em RFC técnica
- não entrar em desenho detalhado de arquitetura, jobs ou serviços
- não omitir origem da informação quando a entrega depender de dados
- não listar regra vaga sem condição observável
- não marcar como validado algo que não veio do material de origem
- não esconder perguntas em aberto só para deixar o documento “fechado”

## Critério de pronto

Não considerar o PRD pronto se faltar:

- problema claro
- objetivo da entrega
- escopo e fora de escopo
- origem da informação, quando aplicável
- chaves, variáveis ou artefatos esperados
- regras de negócio
- critérios de aceite
- métricas
- dependências
- perguntas em aberto ou declaração explícita de ausência
