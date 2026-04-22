# Prompt de Implementação - H-06 Testes de Contrato, Integração e Reprocessamento

## 1. Objetivo da História

Executar e formalizar a estratégia de validação final do pipeline cobrindo contrato, integração ponta a ponta e reprocessamento.
O objetivo é comprovar que o schema final está correto, que os 3 Glue Jobs e a Step Function funcionam de forma integrada e que reprocessar a mesma competência não gera duplicidade lógica.

## 2. Contexto

Esta história depende de H-02, H-03, H-04 e H-05, porque ela valida a implementação consolidada do pipeline.
Ela é a história final da RFC e funciona como gate técnico para considerar a solução pronta.

## 3. Entradas Obrigatórias

### 3.1 Tabelas / arquivos de origem

| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| `tb_contrato_imobiliario_origem` | tabela | Glue Catalog | massa de entrada para testes de validação e cálculo |
| `dataset_calculado_h03` | dataset/tabela intermediária | saída do Job 2 | validar cálculo intermediário |
| `tb_margem_gerencial_contrato_imobiliario` | tabela | Glue Catalog | validar schema e publicação final |
| `state_machine_execution` | execução | Step Functions | validar fluxo ponta a ponta |

### 3.2 Campos por origem

| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| `tb_contrato_imobiliario_origem` | `cd_contrato` | string | Sim | cenário de chave válida e inválida |
| `tb_contrato_imobiliario_origem` | `dt_vencimento_final` | date/timestamp | Sim | teste de prazo remanescente |
| `tb_contrato_imobiliario_origem` | `vl_garantia_imovel` | decimal/double | Sim | teste de `vl_ltv_atual = null` |
| `tb_contrato_imobiliario_origem` | `vl_saldo_devedor_atual` | decimal/double | Sim | validação de fórmulas |
| `tb_margem_gerencial_contrato_imobiliario` | todos os campos do contrato final | schema final | Sim | teste de contrato da saída |
| `state_machine_execution` | `dt_referencia` | date/string | Sim | teste de reprocessamento da mesma competência |
| `state_machine_execution` | `execution_status` | string | Sim | evidência de integração ponta a ponta |

## 4. Regras que devem virar código

| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | As 10 variáveis devem ter teste unitário ou cenário de validação equivalente | cobrir as fórmulas da RFC e regras complementares | campos de cálculo da origem |
| RN02 | O pipeline completo deve ser executável ponta a ponta | rodar Step Function + 3 Glue Jobs com massa controlada | `state_machine_execution`, tabelas de origem e saída |
| RN03 | O contrato da tabela final deve ser exato | validar nomes e tipos das colunas finais publicadas | `tb_margem_gerencial_contrato_imobiliario` |
| RN04 | Reprocessamento da mesma competência não pode duplicar dados | executar novamente a mesma `dt_referencia` e validar sobrescrita lógica | `state_machine_execution.dt_referencia`, tabela final |
| RN05 | Casos obrigatórios da RFC devem ter evidência | validar contrato válido, garantia inválida, contrato vencido, chave nula e reexecução do mês | massa de teste controlada |

## 5. Fluxo de Implementação

1. Preparar massa de teste representativa com cenários válidos e inválidos.
2. Executar testes unitários das fórmulas e regras complementares.
3. Rodar a Step Function completa para uma `dt_referencia` de teste.
4. Validar o dataset calculado intermediário.
5. Validar o schema e o conteúdo da tabela final publicada.
6. Reexecutar a mesma `dt_referencia`.
7. Confirmar ausência de duplicidade lógica na tabela final.
8. Consolidar evidências de sucesso e falha por cenário.

## 6. Saída Esperada

### 6.1 Dataset/tabela/artefato de saída

| Campo | Origem | Regra |
|---|---|---|
| `resultado_testes_unitarios` | derivado | evidência das fórmulas e regras complementares |
| `resultado_integracao` | derivado | evidência da execução ponta a ponta |
| `resultado_contrato` | derivado | confirmação de schema final aderente |
| `resultado_reprocessamento` | derivado | confirmação de sobrescrita sem duplicidade |
| `dt_referencia` | execução de teste | competência validada |

### 6.2 Critérios de aceite técnicos

- deve existir evidência de cobertura das 10 variáveis
- a Step Function deve concluir com sucesso no cenário feliz
- a tabela final deve aderir exatamente ao schema acordado
- reprocessar a mesma competência não pode duplicar a saída
- os casos obrigatórios da RFC devem estar cobertos pela suíte final

## 7. Restrições

- não alterar regras de negócio apenas para acomodar testes
- não considerar pronto sem validação de reprocessamento
- não aceitar contrato final divergente do definido na RFC
- premissa: existe ambiente de homologação capaz de executar a Step Function e os 3 Glue Jobs com massa representativa

## 8. Entregáveis de código

- suíte de testes unitários das regras de cálculo
- suíte de integração ponta a ponta do pipeline
- validação automatizada do schema final
- roteiro ou automação de reprocessamento da mesma competência
- evidências consolidadas para aprovação técnica

## 9. Estimativa

| Bloco | Esforço |
|---|---|
| preparação da massa de teste | 0,5 dia |
| testes unitários das fórmulas | 0,5 dia |
| teste de integração ponta a ponta | 0,5 dia |
| teste de contrato final | 0,25 dia |
| teste de reprocessamento | 0,25 dia |
| total | 2 dias úteis |

**Tamanho:** `M`
