# Prompt de Implementação - H-04 Publicação da Tabela Final

## 1. Objetivo da História

Implementar o `Glue Job 3` para publicar a tabela final `tb_margem_gerencial_contrato_imobiliario`.
O job deve escrever a saída com uma linha por contrato por `dt_referencia`, garantir o registro no Glue Catalog e suportar reprocessamento da mesma competência com sobrescrita lógica da partição.

## 2. Contexto

Esta história depende de H-03, que produz o dataset calculado.
Ela habilita H-05 e H-06, porque a observabilidade e os testes finais dependem da publicação efetiva da saída final.

Na RFC, a publicação é a etapa que torna consumível o resultado do pipeline.

## 3. Entradas Obrigatórias

### 3.1 Tabelas / arquivos de origem

| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| `dataset_calculado_h03` | dataset/tabela intermediária | saída do `Glue Job 2` | base calculada com as chaves e 10 variáveis |
| `execution_input` | payload | Step Function / Glue args | informar `dt_referencia` da competência |
| `tb_margem_gerencial_contrato_imobiliario` | tabela | Glue Catalog | tabela final a ser criada/atualizada |

### 3.2 Campos por origem

| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| `execution_input` | `dt_referencia` | date/string normalizável para data | Sim | partição lógica de publicação |
| `dataset_calculado_h03` | `cd_contrato` | string | Sim | chave da saída final |
| `dataset_calculado_h03` | `cd_sistema_origem` | string | Sim | chave da saída final |
| `dataset_calculado_h03` | `nr_proposta` | string | Sim | chave da saída final |
| `dataset_calculado_h03` | `nr_cpf_cnpj_cliente` | string | Sim | chave da saída final |
| `dataset_calculado_h03` | `dt_referencia` | date | Sim | partição lógica da saída |
| `dataset_calculado_h03` | `vl_saldo_medio` | decimal(18,2) | Sim | coluna final |
| `dataset_calculado_h03` | `vl_margem_liquida` | decimal(18,2) | Sim | coluna final |
| `dataset_calculado_h03` | `vl_receita_juros` | decimal(18,2) | Sim | coluna final |
| `dataset_calculado_h03` | `vl_custo_funding` | decimal(18,2) | Sim | coluna final |
| `dataset_calculado_h03` | `vl_custo_operacional` | decimal(18,2) | Sim | coluna final |
| `dataset_calculado_h03` | `vl_perda_esperada` | decimal(18,2) | Sim | coluna final |
| `dataset_calculado_h03` | `vl_receita_tarifas` | decimal(18,2) | Sim | coluna final |
| `dataset_calculado_h03` | `vl_resultado_bruto` | decimal(18,2) | Sim | coluna final |
| `dataset_calculado_h03` | `vl_ltv_atual` | decimal(18,6) | Sim | coluna final |
| `dataset_calculado_h03` | `vl_prazo_remanescente_meses` | int | Sim | coluna final |

## 4. Regras que devem virar código

| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | A tabela final deve conter exatamente as chaves e 10 variáveis definidas na RFC | escrever o schema final sem criar campos adicionais | todos os campos do `dataset_calculado_h03` |
| RN02 | A granularidade da saída é uma linha por contrato por mês de referência | publicar mantendo unicidade lógica por contrato e `dt_referencia` | chaves + `dt_referencia` |
| RN03 | A publicação deve ocorrer por `dt_referencia` | gravar usando partição lógica por competência | `execution_input.dt_referencia` ou `dataset_calculado_h03.dt_referencia` |
| RN04 | O registro da tabela no Glue Catalog é obrigatório | só considerar a publicação concluída após garantir a tabela no catálogo | `tb_margem_gerencial_contrato_imobiliario` |
| RN05 | Reprocessamento da mesma competência deve sobrescrever a partição | substituir logicamente a partição do mês reprocessado, sem duplicidade | `dt_referencia` |
| RN06 | Falha na publicação impede status de sucesso | retornar falha quando escrita ou registro no catálogo falharem | resultado da escrita/catalogação |

## 5. Fluxo de Implementação

1. Ler `dt_referencia` dos argumentos do job.
2. Ler o `dataset_calculado_h03`.
3. Validar que o schema de entrada contém as colunas finais esperadas.
4. Preparar a estrutura de escrita da tabela `tb_margem_gerencial_contrato_imobiliario`.
5. Gravar os dados da competência em partição lógica por `dt_referencia`.
6. Garantir o registro ou atualização da tabela no Glue Catalog.
7. Retornar status lógico da publicação e a partição publicada.

## 6. Saída Esperada

### 6.1 Dataset/tabela/artefato de saída

| Campo | Origem | Regra |
|---|---|---|
| `cd_contrato` | `dataset_calculado_h03.cd_contrato` | cópia direta |
| `cd_sistema_origem` | `dataset_calculado_h03.cd_sistema_origem` | cópia direta |
| `nr_proposta` | `dataset_calculado_h03.nr_proposta` | cópia direta |
| `nr_cpf_cnpj_cliente` | `dataset_calculado_h03.nr_cpf_cnpj_cliente` | cópia direta |
| `dt_referencia` | `dataset_calculado_h03.dt_referencia` | cópia direta / partição lógica |
| `vl_saldo_medio` | `dataset_calculado_h03.vl_saldo_medio` | cópia direta |
| `vl_margem_liquida` | `dataset_calculado_h03.vl_margem_liquida` | cópia direta |
| `vl_receita_juros` | `dataset_calculado_h03.vl_receita_juros` | cópia direta |
| `vl_custo_funding` | `dataset_calculado_h03.vl_custo_funding` | cópia direta |
| `vl_custo_operacional` | `dataset_calculado_h03.vl_custo_operacional` | cópia direta |
| `vl_perda_esperada` | `dataset_calculado_h03.vl_perda_esperada` | cópia direta |
| `vl_receita_tarifas` | `dataset_calculado_h03.vl_receita_tarifas` | cópia direta |
| `vl_resultado_bruto` | `dataset_calculado_h03.vl_resultado_bruto` | cópia direta |
| `vl_ltv_atual` | `dataset_calculado_h03.vl_ltv_atual` | cópia direta |
| `vl_prazo_remanescente_meses` | `dataset_calculado_h03.vl_prazo_remanescente_meses` | cópia direta |

### 6.2 Critérios de aceite técnicos

- a tabela final deve refletir exatamente o contrato de schema da RFC
- a tabela deve estar registrada no Glue Catalog ao fim da execução
- reprocessar a mesma competência não pode criar duplicidade lógica
- falha de escrita ou de catalogação deve resultar em falha do job

## 7. Restrições

- não recalcular variáveis nesta etapa
- não alterar nomes ou tipos sugeridos pela RFC sem aceite explícito
- não considerar sucesso sem registro no Glue Catalog
- premissa: existe uma área intermediária produzida por H-03 apta a ser lida pelo `Glue Job 3`

## 8. Entregáveis de código

- implementação do `Glue Job 3`
- escrita da tabela final por `dt_referencia`
- garantia de registro da tabela no Glue Catalog
- tratamento de reprocessamento com sobrescrita lógica
- testes de contrato, integração de escrita e reprocessamento

## 9. Estimativa

| Bloco | Esforço |
|---|---|
| leitura do dataset calculado | 0,25 dia |
| validação de schema final | 0,25 dia |
| persistência da tabela final | 0,5 dia |
| registro no Glue Catalog | 0,5 dia |
| testes de reprocessamento e contrato | 0,5 dia |
| total | 2 dias úteis |

**Tamanho:** `M`
