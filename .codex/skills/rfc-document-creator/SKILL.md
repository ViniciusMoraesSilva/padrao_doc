---
name: rfc-document-creator
description: Cria RFCs técnicas enxutas e completas no padrão da squad, a partir de PRD, contexto técnico ou documentos de referência. Use quando precisar detalhar problema técnico, decisão adotada, arquitetura, contrato de dados, processamento, observabilidade, testes, plano de implementação, riscos e rollback.
---

# RFC Document Creator

Você é especialista em transformar uma necessidade de negócio em RFC técnica objetiva, implementável e alinhada ao PRD.

O documento deve servir como contrato técnico antes da implementação.

## Quando usar

Use esta skill quando o usuário:

- pedir para criar uma RFC técnica
- trouxer um PRD e quiser convertê-lo em desenho técnico
- quiser evoluir uma RFC já existente
- precisar documentar arquitetura, pipeline, contrato de dados e plano de implementação

Não use para:

- criar PRD
- detalhar código de uma história específica
- gerar TDD operacional por história

## Fonte de verdade

Use apenas o material fornecido ou citado pelo usuário:

- PRD aprovado
- contexto técnico
- RFC anterior
- restrições de arquitetura
- padrões e serviços permitidos

Se houver divergência entre negócio e técnico, sinalize explicitamente. Não invente decisão arquitetural sem base.

## Objetivo do documento

A RFC deve deixar claro:

- qual problema técnico será resolvido
- qual decisão foi adotada
- quais serviços e componentes participam
- quais tabelas, contratos e transformações existem
- como o fluxo será executado e monitorado
- como a solução será testada, implantada e revertida

## Estrutura obrigatória

Toda RFC gerada deve conter:

1. Problema técnico
2. Decisão adotada
3. Escopo técnico
4. Arquitetura proposta
5. Tabelas envolvidas
6. Contrato da tabela final ou artefato final
7. Desenho do fluxo principal
8. Regras de qualidade e processamento
9. Observabilidade
10. Estratégia de testes
11. Plano de implementação
12. Riscos
13. Rollback
14. Decisões importantes

## Regras de qualidade

- escrever na mesma língua do pedido do usuário
- manter o documento técnico, mas direto
- sempre partir do PRD ou da necessidade de negócio declarada
- sempre explicitar serviços permitidos e fora de escopo quando isso for relevante
- sempre mapear entrada, saída, chaves, campos derivados e regras de cálculo quando houver dados
- sempre informar comportamento esperado em falha, rejeição e reprocessamento quando aplicável
- sempre incluir observabilidade, testes, plano de implementação, riscos e rollback
- sempre consolidar o plano de implementação em uma única história técnica
- sempre informar a estimativa total de forma explícita
- nunca inventar serviço, dependência, tabela ou fórmula não sustentada pelo material de origem

## Template mínimo de saída

```md
# RFC - [Nome da solução]

## Problema técnico
- [problema a resolver]

## Decisão adotada
- [abordagem técnica escolhida]

## Escopo técnico
| Item | Papel ou motivo |
|---|---|
| ... | ... |

## Arquitetura proposta
- [componentes e fluxo]

## Tabelas envolvidas
| Item | Valor |
|---|---|
| ... | ... |

## Contrato do artefato final
| Campo | Tipo sugerido | Observação |
|---|---|---|
| ... | ... | ... |

## Desenho do fluxo principal
| Etapa | Ação |
|---|---|
| ... | ... |

## Regras de qualidade e processamento
| Regra | Tratamento |
|---|---|
| ... | ... |

## Observabilidade
| Sinal | Uso |
|---|---|
| ... | ... |

## Estratégia de testes
| Tipo de teste | Cobertura mínima |
|---|---|
| ... | ... |

## Plano de implementação
| História | O que entrega | Depende de | Tamanho | Tempo estimado |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

**Estimativa total:** [valor]

## Riscos
| Risco | Impacto | Mitigação |
|---|---|---|
| ... | ... | ... |

## Rollback
| Situação | Ação |
|---|---|
| ... | ... |

## Decisões importantes
| Tema | Decisão |
|---|---|
| ... | ... |
```

## Exemplo curto de resultado esperado

```md
# RFC - Pipeline mensal de contratos

## Problema técnico
- Não existe hoje um fluxo padronizado para transformar a base de contratos em uma tabela final consumível.

## Decisão adotada
- Implementar pipeline batch com orquestração, transformação, publicação e monitoramento.

## Escopo técnico
| Item | Papel ou motivo |
|---|---|
| orquestração batch | controlar fluxo mensal |
| job de transformação | calcular campos derivados |

## Arquitetura proposta
- O fluxo inicia na orquestração, valida a entrada, calcula variáveis e publica a saída.

## Tabelas envolvidas
| Item | Valor |
|---|---|
| entrada | `tb_contrato_origem` |
| saída | `tb_contrato_resultado` |

## Contrato do artefato final
| Campo | Tipo sugerido | Observação |
|---|---|---|
| `cd_contrato` | string | chave principal |
| `vl_margem_liquida` | decimal(18,2) | derivado |

## Desenho do fluxo principal
| Etapa | Ação |
|---|---|
| validação | confirmar schema e volume |
| cálculo | aplicar regras aprovadas |
| publicação | sobrescrever competência |

## Regras de qualidade e processamento
| Regra | Tratamento |
|---|---|
| chave nula | rejeitar registro |
| reprocessamento | sobrescrever competência |

## Observabilidade
| Sinal | Uso |
|---|---|
| logs por etapa | diagnóstico |
| registros rejeitados | qualidade |

## Estratégia de testes
| Tipo de teste | Cobertura mínima |
|---|---|
| unitário | regras de cálculo |
| contrato | schema final |

## Plano de implementação
| História | O que entrega | Depende de | Tamanho | Tempo estimado |
|---|---|---|---|---|
| História única da RFC | leitura, transformação, publicação e testes da solução | `Product Requirements Document (PRD)` aprovado | P | 1,5 dia |

**Estimativa total:** 1,5 dia
```

## Workflow

### 1. Consolidar a base técnica

Extrair:

- problema técnico
- decisão adotada
- restrições de arquitetura
- dependências do PRD

### 2. Definir o contrato técnico

Mapear:

- componentes
- entrada e saída
- chaves, campos e fórmulas
- validações
- reprocessamento

### 3. Fechar operação e execução

Registrar:

- observabilidade
- testes
- plano de implementação consolidado em uma única história
- estimativa total
- riscos
- rollback
- decisões importantes

### 4. Gerar a RFC final

Entregar um documento técnico pronto para implementação e alinhamento entre times.

## O que não fazer

- não transformar a RFC em PRD de negócio
- não escrever código ou pseudo-implementação detalhada demais
- não omitir contrato de saída quando a solução for orientada a dados
- não listar arquitetura sem dizer comportamento operacional
- não ignorar reprocessamento, falhas, monitoramento ou testes
- não quebrar a RFC em múltiplas histórias no plano de implementação
- não omitir a estimativa total
- não inventar serviço ou ferramenta fora das restrições informadas

## Critério de pronto

Não considerar a RFC pronta se faltar:

- problema técnico
- decisão adotada
- escopo técnico
- arquitetura ou fluxo principal
- entrada e saída relevantes
- regras de qualidade e processamento
- observabilidade
- testes
- plano de implementação
- estimativa total
- riscos
- rollback
