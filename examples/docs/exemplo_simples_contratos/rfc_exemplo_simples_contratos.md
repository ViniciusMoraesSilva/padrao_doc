# RFC - Exemplo simples de transformação de contratos

## Problema técnico
- A área de acompanhamento comercial precisa receber uma visão padronizada de contratos para análises recorrentes.
- O `Product Requirements Document (PRD)` já definiu a origem da informação, a granularidade esperada e os campos que o negócio precisa receber.
- Para este exemplo, a necessidade técnica é transformar esse direcionamento de negócio em um fluxo simples de leitura, cálculo e publicação.

## Decisão adotada
- Implementar um fluxo batch simples com `job AWS Glue` para ler `tb_contratos_origem`, respeitar a granularidade de 1 linha por contrato, validar os campos mínimos, calcular dois novos campos e publicar o resultado em `tb_contratos_saida`.

## Escopo técnico
| Item | Papel ou motivo |
|---|---|
| `job AWS Glue` | Executar a leitura, validação, transformação e publicação da saída |
| Leitura da tabela de entrada | Buscar `chave_contrato`, `valor_1`, `valor_2` e `valor_3` conforme definido pelo negócio |
| Validação mínima | Garantir presença de chave e campos necessários |
| Transformação | Aplicar as fórmulas de cálculo |
| Publicação | Escrever a tabela final com estrutura padronizada |

## Arquitetura proposta
- Um `job AWS Glue` único realiza leitura, validação, cálculo e publicação.
- O fluxo é simples e linear, sem dependências externas adicionais.
- A solução foi mantida enxuta para facilitar entendimento na apresentação.

## Tabelas envolvidas
| Item | Valor |
|---|---|
| Tabela de entrada | `tb_contratos_origem` |
| Tabela de saída | `tb_contratos_saida` |
| Chave principal | `chave_contrato` |
| Granularidade da saída | 1 linha por contrato |
| Time dono da informação de origem | Time dono da informação |

## Contrato do artefato final
| Campo | Tipo sugerido | Observação |
|---|---|---|
| `chave_contrato` | string | chave principal do contrato |
| `valor_1` | decimal(18,2) | campo vindo da origem |
| `valor_2` | decimal(18,2) | campo vindo da origem |
| `valor_3` | decimal(18,2) | campo vindo da origem |
| `valor_calculado_1` | decimal(18,2) | `valor_1 * valor_2` |
| `valor_calculado_2` | decimal(18,2) | `valor_3 / valor_2`, ou nulo quando `valor_2 = 0` |

## Desenho do fluxo principal
| Etapa | Ação |
|---|---|
| Inicialização | Executar o `job AWS Glue` responsável pelo processamento |
| Leitura | Ler `tb_contratos_origem` |
| Validação | Confirmar `chave_contrato`, `valor_1`, `valor_2` e `valor_3` para viabilizar aplicação de RN01 e RN02 do PRD |
| Cálculo 1 | Criar `valor_calculado_1 = valor_1 * valor_2` conforme RN03 do PRD |
| Cálculo 2 | Criar `valor_calculado_2 = valor_3 / valor_2` conforme RN04 do PRD |
| Tratamento de exceção | Quando `valor_2 = 0`, preencher `valor_calculado_2` com nulo conforme RN05 do PRD |
| Publicação | Escrever `tb_contratos_saida` garantindo o contrato final previsto em RN06 do PRD |

## Rastreabilidade das regras de negócio
| Regra do PRD | Aplicação técnica na RFC |
|---|---|
| RN01 | Manter 1 linha por contrato no processamento e na publicação final |
| RN02 | Ler e publicar `chave_contrato`, `valor_1`, `valor_2` e `valor_3` |
| RN03 | Calcular `valor_calculado_1 = valor_1 * valor_2` no `job AWS Glue` |
| RN04 | Calcular `valor_calculado_2 = valor_3 / valor_2` no `job AWS Glue` |
| RN05 | Quando o segundo cálculo não puder ser realizado por `valor_2 = 0`, publicar `valor_calculado_2` como nulo |
| RN06 | Garantir que `tb_contratos_saida` contenha exatamente os 6 campos esperados pelo negócio |

## Regras de qualidade e processamento
| Regra | Tratamento |
|---|---|
| `chave_contrato` ausente | rejeitar o registro sem violar RN01, pois não há contrato válido para compor a saída |
| `valor_1`, `valor_2` ou `valor_3` ausente | rejeitar o registro por impedir a aplicação de RN02, RN03 e RN04 |
| `valor_2 = 0` | manter o registro e publicar `valor_calculado_2` como nulo, aplicando RN05 |
| Reprocessamento | sobrescrever a saída do exemplo sem duplicar registros |

## Observabilidade
| Sinal | Uso |
|---|---|
| Status de execução do `job AWS Glue` | acompanhar sucesso ou falha do processamento |
| Quantidade de registros lidos | acompanhar volume de entrada |
| Quantidade de registros publicados | acompanhar volume de saída |
| Quantidade de registros rejeitados | acompanhar qualidade do processamento |
| Quantidade de registros com `valor_calculado_2` nulo por divisão inválida | acompanhar ocorrência da regra de exceção |

## Estratégia de testes
| Tipo de teste | Cobertura mínima |
|---|---|
| Unitário | validar as duas fórmulas |
| Contrato | validar schema final da tabela de saída |
| Integração | validar leitura, cálculo e publicação |
| Exceção | validar tratamento de `valor_2 = 0` |

## Plano de implementação
| História | O que entrega | Depende de | Tamanho | Tempo estimado |
|---|---|---|---|---|
| História única da RFC | configuração e execução do `job AWS Glue` para leitura da origem, validação mínima, cálculo dos campos novos, publicação da saída e testes principais | `Product Requirements Document (PRD)` aprovado | P | 1,5 dia |

**Estimativa total:** 1,5 dia

## Riscos
| Risco | Impacto | Mitigação |
|---|---|---|
| Campo de origem ausente | Médio | validar schema antes do cálculo |
| Divisão por zero | Baixo | preencher `valor_calculado_2` com nulo |
| Saída com estrutura incompleta | Médio | validar contrato final antes da publicação |

## Rollback
| Situação | Ação |
|---|---|
| Falha no cálculo | não publicar a saída |
| Falha na publicação | manter a tabela anterior sem alteração |
| Erro estrutural identificado após publicação | restaurar a versão anterior da saída do exemplo |

## Decisões importantes
| Tema | Decisão |
|---|---|
| Tipo de processamento | batch simples com `job AWS Glue` |
| Tratamento de divisão por zero | publicar nulo em `valor_calculado_2` |
| Estrutura da saída | manter campos originais + campos calculados |
| Complexidade da solução | manter o fluxo enxuto e didático |
