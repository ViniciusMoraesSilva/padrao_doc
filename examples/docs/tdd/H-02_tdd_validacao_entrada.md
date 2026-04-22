# Prompt de Implementação - H-02 Validação da Entrada

## 1. Objetivo da História

Implementar o `Glue Job 1` responsável por validar a tabela de entrada `tb_contrato_imobiliario_origem`.
O job deve bloquear a continuidade do pipeline quando a tabela não existir, quando o schema mínimo estiver incompleto, quando tipos essenciais estiverem incompatíveis ou quando a carga vier vazia.

## 2. Contexto

Esta história depende de H-01, que cria a orquestração da Step Function.
Ela habilita H-03, porque o cálculo só pode ocorrer após a entrada estar estruturalmente válida.

Segundo a RFC, a origem deve estar no Glue Catalog e conter os campos aprovados pelo negócio para o cálculo da margem gerencial.

## 3. Entradas Obrigatórias

### 3.1 Tabelas / arquivos de origem

| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| `tb_contrato_imobiliario_origem` | tabela | Glue Catalog | base de contratos imobiliários a ser validada |
| `execution_input` | payload | Step Function / Glue args | informar a `dt_referencia` e contexto da execução |

### 3.2 Campos por origem

| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| `execution_input` | `dt_referencia` | date/string normalizável para data | Sim | rastreabilidade da execução |
| `tb_contrato_imobiliario_origem` | `cd_contrato` | string | Sim | chave obrigatória |
| `tb_contrato_imobiliario_origem` | `cd_sistema_origem` | string | Sim | chave obrigatória |
| `tb_contrato_imobiliario_origem` | `nr_proposta` | string | Sim | chave obrigatória |
| `tb_contrato_imobiliario_origem` | `nr_cpf_cnpj_cliente` | string | Sim | campo obrigatório validado pela RFC |
| `tb_contrato_imobiliario_origem` | `dt_contratacao` | date/timestamp | Sim | campo obrigatório validado pela RFC |
| `tb_contrato_imobiliario_origem` | `dt_vencimento_final` | date/timestamp | Sim | campo obrigatório para cálculo posterior |
| `tb_contrato_imobiliario_origem` | `vl_saldo_devedor_atual` | decimal/double | Sim | insumo de cálculo |
| `tb_contrato_imobiliario_origem` | `vl_parcela_atual` | decimal/double | Sim | insumo de cálculo |
| `tb_contrato_imobiliario_origem` | `vl_taxa_juros_aa` | decimal/double | Sim | insumo de cálculo |
| `tb_contrato_imobiliario_origem` | `vl_garantia_imovel` | decimal/double | Sim | insumo de cálculo |
| `tb_contrato_imobiliario_origem` | `vl_custo_funding_aa` | decimal/double | Sim | insumo de cálculo |
| `tb_contrato_imobiliario_origem` | `pc_inadimplencia_modelo` | decimal/double | Sim | insumo de cálculo |
| `tb_contrato_imobiliario_origem` | `pc_custo_operacional` | decimal/double | Sim | insumo de cálculo |

## 4. Regras que devem virar código

| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | A tabela de entrada deve existir no Glue Catalog | falhar o job quando `tb_contrato_imobiliario_origem` não existir | `tb_contrato_imobiliario_origem` |
| RN02 | Todos os campos mínimos obrigatórios devem existir | validar schema por nome de coluna antes de prosseguir | todos os campos listados na seção 3.2 |
| RN03 | Tipos essenciais devem ser compatíveis com o cálculo | validar que datas e numéricos estejam em tipos conversíveis/compatíveis; falhar quando incompatíveis | `dt_contratacao`, `dt_vencimento_final`, `vl_*`, `pc_*` |
| RN04 | A quantidade lida deve ser maior que zero | falhar o job quando a leitura da tabela retornar zero registros | `tb_contrato_imobiliario_origem` |
| RN05 | Registros com chave obrigatória nula devem ser contabilizados como inválidos estruturais | contar registros inválidos quando qualquer chave obrigatória estiver nula | `cd_contrato`, `cd_sistema_origem`, `nr_proposta` |
| RN06 | A saída do job deve informar status lógico para a Step Function | retornar `status_validacao`, `quantidade_lida`, `quantidade_invalidos` e `motivo_falha` quando aplicável | resultado do job |

## 5. Fluxo de Implementação

1. Ler `dt_referencia` dos argumentos da execução.
2. Ler o metadata da tabela `tb_contrato_imobiliario_origem` no Glue Catalog.
3. Validar a existência da tabela.
4. Validar a presença das colunas obrigatórias.
5. Validar compatibilidade de tipos das colunas essenciais.
6. Ler a tabela e contar `quantidade_lida`.
7. Falhar quando `quantidade_lida = 0`.
8. Contar `quantidade_invalidos` para registros com chave obrigatória nula.
9. Registrar logs estruturados com a competência e o resumo da validação.
10. Retornar o payload lógico para a Step Function.

## 6. Saída Esperada

### 6.1 Dataset/tabela/artefato de saída

| Campo | Origem | Regra |
|---|---|---|
| `status_validacao` | derivado | `SUCESSO` quando todas as validações passarem; `FALHA` caso contrário |
| `quantidade_lida` | derivado da leitura da tabela | total de registros lidos da origem |
| `quantidade_invalidos` | derivado | total de registros com falha estrutural de chave obrigatória |
| `motivo_falha` | derivado | preenchido quando a validação falhar |
| `dt_referencia` | `execution_input.dt_referencia` | cópia direta para rastreabilidade |

### 6.2 Critérios de aceite técnicos

- o job deve falhar se a tabela não existir
- o job deve falhar se qualquer campo obrigatório estiver ausente
- o job deve falhar se tipos essenciais estiverem incompatíveis
- o job deve falhar se a tabela estiver vazia
- o job deve retornar contagem de inválidos estruturais para consumo operacional

## 7. Restrições

- não aplicar fórmulas de cálculo nesta etapa
- não publicar dados de saída nesta etapa
- não inventar colunas fora das aprovadas na RFC
- premissa: a política de mascaramento de `nr_cpf_cnpj_cliente` será definida antes do go-live, como indicado na RFC

## 8. Entregáveis de código

- implementação do `Glue Job 1`
- leitura da tabela via Glue Catalog
- validação de schema e tipos obrigatórios
- contagem de volume lido e inválidos estruturais
- payload de retorno padronizado para a Step Function
- testes para tabela inexistente, schema inválido, carga vazia e chave nula

## 9. Estimativa

| Bloco | Esforço |
|---|---|
| leitura do catálogo e metadata | 0,5 dia |
| validação de schema e tipos | 0,5 dia |
| contagem de volume e inválidos | 0,25 dia |
| retorno padronizado para Step Function | 0,25 dia |
| testes | 0,5 dia |
| total | 2 dias úteis |

**Tamanho:** `M`
