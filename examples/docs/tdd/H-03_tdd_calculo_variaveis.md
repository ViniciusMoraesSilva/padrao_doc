# Prompt de Implementação - H-03 Cálculo das 10 Variáveis

## 1. Objetivo da História

Implementar o `Glue Job 2` para calcular as 10 variáveis de valor por contrato imobiliário.
O job deve ler `tb_contrato_imobiliario_origem`, padronizar os dados, aplicar exatamente as fórmulas técnicas da RFC, rejeitar registros inválidos e produzir uma base com uma linha por contrato válido e por `dt_referencia`.

## 2. Contexto

Esta história depende de H-02, que garante a validade estrutural da entrada.
Ela habilita H-04, que publicará a tabela final no Glue Catalog.

O coração funcional da RFC está nesta etapa, porque aqui as regras aprovadas no PRD viram código operacional.

## 3. Entradas Obrigatórias

### 3.1 Tabelas / arquivos de origem

| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| `tb_contrato_imobiliario_origem` | tabela | Glue Catalog | base validada com os contratos a calcular |
| `execution_input` | payload | Step Function / Glue args | informar `dt_referencia` da competência |

### 3.2 Campos por origem

| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| `execution_input` | `dt_referencia` | date/string normalizável para data | Sim | cálculo de `vl_prazo_remanescente_meses` e partição lógica |
| `tb_contrato_imobiliario_origem` | `cd_contrato` | string | Sim | chave da saída |
| `tb_contrato_imobiliario_origem` | `cd_sistema_origem` | string | Sim | chave da saída |
| `tb_contrato_imobiliario_origem` | `nr_proposta` | string | Sim | chave da saída |
| `tb_contrato_imobiliario_origem` | `nr_cpf_cnpj_cliente` | string | Sim | chave da saída |
| `tb_contrato_imobiliario_origem` | `dt_vencimento_final` | date/timestamp | Sim | cálculo de prazo remanescente |
| `tb_contrato_imobiliario_origem` | `vl_saldo_devedor_atual` | decimal/double | Sim | base para `vl_saldo_medio`, `vl_receita_juros`, `vl_custo_funding`, `vl_custo_operacional`, `vl_perda_esperada`, `vl_ltv_atual` |
| `tb_contrato_imobiliario_origem` | `vl_parcela_atual` | decimal/double | Sim | base para `vl_receita_tarifas` |
| `tb_contrato_imobiliario_origem` | `vl_taxa_juros_aa` | decimal/double | Sim | base para `vl_receita_juros` |
| `tb_contrato_imobiliario_origem` | `vl_garantia_imovel` | decimal/double | Sim | base para `vl_ltv_atual` |
| `tb_contrato_imobiliario_origem` | `vl_custo_funding_aa` | decimal/double | Sim | base para `vl_custo_funding` |
| `tb_contrato_imobiliario_origem` | `pc_inadimplencia_modelo` | decimal/double | Sim | base para `vl_perda_esperada` |
| `tb_contrato_imobiliario_origem` | `pc_custo_operacional` | decimal/double | Sim | base para `vl_custo_operacional` |

## 4. Regras que devem virar código

| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | `vl_saldo_medio` na V1 é igual ao saldo devedor atual | criar `vl_saldo_medio = vl_saldo_devedor_atual` | `vl_saldo_devedor_atual` |
| RN02 | Receita de juros mensal | criar `vl_receita_juros = vl_saldo_medio * (vl_taxa_juros_aa / 12)` | `vl_saldo_medio`, `vl_taxa_juros_aa` |
| RN03 | Custo de funding mensal | criar `vl_custo_funding = vl_saldo_medio * (vl_custo_funding_aa / 12)` | `vl_saldo_medio`, `vl_custo_funding_aa` |
| RN04 | Custo operacional | criar `vl_custo_operacional = vl_saldo_medio * pc_custo_operacional` | `vl_saldo_medio`, `pc_custo_operacional` |
| RN05 | Perda esperada | criar `vl_perda_esperada = vl_saldo_medio * pc_inadimplencia_modelo` | `vl_saldo_medio`, `pc_inadimplencia_modelo` |
| RN06 | Receita de tarifas | criar `vl_receita_tarifas = vl_parcela_atual * 0.002` | `vl_parcela_atual` |
| RN07 | Resultado bruto | criar `vl_resultado_bruto = vl_receita_juros + vl_receita_tarifas - vl_custo_funding` | `vl_receita_juros`, `vl_receita_tarifas`, `vl_custo_funding` |
| RN08 | LTV atual | criar `vl_ltv_atual = vl_saldo_devedor_atual / vl_garantia_imovel`; quando `vl_garantia_imovel <= 0`, retornar `null` | `vl_saldo_devedor_atual`, `vl_garantia_imovel` |
| RN09 | Prazo remanescente em meses | calcular a diferença em meses entre `dt_referencia` e `dt_vencimento_final`; quando `dt_vencimento_final < dt_referencia`, retornar `0` | `dt_referencia`, `dt_vencimento_final` |
| RN10 | Margem líquida | criar `vl_margem_liquida = vl_resultado_bruto - vl_perda_esperada - vl_custo_operacional` | `vl_resultado_bruto`, `vl_perda_esperada`, `vl_custo_operacional` |
| RN11 | Registros sem chave obrigatória ou sem insumo mínimo não entram na saída | rejeitar registros inválidos e contabilizar rejeição em log estruturado | `cd_contrato`, `cd_sistema_origem`, `nr_proposta`, `nr_cpf_cnpj_cliente`, insumos de cálculo |

## 5. Fluxo de Implementação

1. Ler `dt_referencia` dos argumentos do job.
2. Ler `tb_contrato_imobiliario_origem`.
3. Padronizar tipos numéricos e datas necessárias ao cálculo.
4. Filtrar ou rejeitar registros sem chave obrigatória ou sem insumo mínimo.
5. Criar `vl_saldo_medio`.
6. Calcular `vl_receita_juros`, `vl_custo_funding`, `vl_custo_operacional`, `vl_perda_esperada` e `vl_receita_tarifas`.
7. Calcular `vl_resultado_bruto`, `vl_ltv_atual`, `vl_prazo_remanescente_meses` e `vl_margem_liquida`.
8. Adicionar `dt_referencia` a cada linha válida.
9. Registrar contadores de processados e rejeitados em log estruturado.
10. Persistir o dataset intermediário para consumo da publicação final.

## 6. Saída Esperada

### 6.1 Dataset/tabela/artefato de saída

| Campo | Origem | Regra |
|---|---|---|
| `cd_contrato` | `tb_contrato_imobiliario_origem.cd_contrato` | cópia direta |
| `cd_sistema_origem` | `tb_contrato_imobiliario_origem.cd_sistema_origem` | cópia direta |
| `nr_proposta` | `tb_contrato_imobiliario_origem.nr_proposta` | cópia direta |
| `nr_cpf_cnpj_cliente` | `tb_contrato_imobiliario_origem.nr_cpf_cnpj_cliente` | cópia direta |
| `dt_referencia` | `execution_input.dt_referencia` | cópia direta |
| `vl_saldo_medio` | derivado | RN01 |
| `vl_receita_juros` | derivado | RN02 |
| `vl_custo_funding` | derivado | RN03 |
| `vl_custo_operacional` | derivado | RN04 |
| `vl_perda_esperada` | derivado | RN05 |
| `vl_receita_tarifas` | derivado | RN06 |
| `vl_resultado_bruto` | derivado | RN07 |
| `vl_ltv_atual` | derivado | RN08 |
| `vl_prazo_remanescente_meses` | derivado | RN09 |
| `vl_margem_liquida` | derivado | RN10 |

### 6.2 Critérios de aceite técnicos

- a saída deve ter uma linha por contrato válido por `dt_referencia`
- as fórmulas devem reproduzir exatamente as regras técnicas da RFC
- `vl_ltv_atual` deve ser `null` quando `vl_garantia_imovel <= 0`
- `vl_prazo_remanescente_meses` deve ser `0` quando `dt_vencimento_final < dt_referencia`
- registros inválidos não podem seguir para a saída

## 7. Restrições

- não criar fórmulas fora das 10 regras técnicas documentadas na RFC
- não usar tabelas auxiliares ou joins não descritos na RFC
- não publicar a tabela final nesta etapa
- premissa: o dataset intermediário calculado será persistido em área interna do pipeline para consumo do `Glue Job 3`, embora a RFC não detalhe o nome físico desse artefato

## 8. Entregáveis de código

- implementação do `Glue Job 2`
- padronização de tipos e nulos
- cálculo das 10 variáveis
- tratamento de rejeição de registros inválidos
- persistência do dataset intermediário calculado
- testes unitários por fórmula e regras complementares

## 9. Estimativa

| Bloco | Esforço |
|---|---|
| leitura e padronização da entrada | 0,5 dia |
| implementação das fórmulas RN01 a RN10 | 1,5 dia |
| regras de rejeição e logs estruturados | 0,5 dia |
| persistência intermediária | 0,5 dia |
| testes | 1 dia |
| total | 4 dias úteis |

**Tamanho:** `G`
