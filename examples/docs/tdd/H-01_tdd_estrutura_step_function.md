# Prompt de Implementação - H-01 Estrutura da Step Function

## 1. Objetivo da História

Implementar a Step Function mensal que orquestra o pipeline de margem gerencial para contratos imobiliários.
O fluxo deve disparar, em sequência, os jobs de validação, cálculo e publicação, sempre propagando `dt_referencia` e encerrando em sucesso ou falha de forma explícita.

## 2. Contexto

Esta é a primeira história da RFC `../rfc/exemplo_rfc_margem.md` e habilita toda a execução do pipeline batch mensal.
A dependência anterior é inexistente. Esta história habilita H-02, H-03 e H-04 ao criar o contrato de orquestração que controlará os Glue Jobs.

A RFC determina:
- uso exclusivo de Step Functions para orquestração
- execução mensal por competência
- interrupção imediata quando validação, cálculo ou publicação falharem

## 3. Entradas Obrigatórias

### 3.1 Tabelas / arquivos / recursos de origem

| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| `state_machine_input` | payload | entrada da execução da Step Function | informar a competência e parâmetros mínimos da execução |
| `Glue Job 1` | job AWS Glue | ambiente AWS | validar a entrada antes do cálculo |
| `Glue Job 2` | job AWS Glue | ambiente AWS | calcular as 10 variáveis de valor |
| `Glue Job 3` | job AWS Glue | ambiente AWS | publicar a tabela final e registrar a saída |

### 3.2 Campos por origem

| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| `state_machine_input` | `dt_referencia` | date/string normalizável para data | Sim | chave da execução mensal e parâmetro propagado para todos os jobs |
| `Glue Job 1` | `status_validacao` | string | Sim | decisão no estado `ChecarValidacao` |
| `Glue Job 1` | `quantidade_lida` | int | Sim | log e rastreabilidade da validação |
| `Glue Job 1` | `quantidade_invalidos` | int | Sim | log e rastreabilidade da validação |
| `Glue Job 1` | `motivo_falha` | string | Não | mensagem de erro quando a validação falhar |
| `Glue Job 2` | `status_calculo` | string | Sim | decisão no estado `ChecarCalculo` |
| `Glue Job 2` | `quantidade_processada` | int | Sim | log e rastreabilidade do cálculo |
| `Glue Job 2` | `quantidade_rejeitada` | int | Sim | log e rastreabilidade do cálculo |
| `Glue Job 2` | `motivo_falha` | string | Não | mensagem de erro quando o cálculo falhar |
| `Glue Job 3` | `status_publicacao` | string | Sim | decisão final de sucesso ou falha |
| `Glue Job 3` | `particao_publicada` | string/date | Sim | evidência da competência publicada |
| `Glue Job 3` | `motivo_falha` | string | Não | mensagem de erro quando a publicação falhar |

## 4. Regras que devem virar código

| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | A execução deve começar sempre pela validação da entrada | criar o estado `ValidarEntrada` que dispara o `Glue Job 1` com `dt_referencia` | `state_machine_input.dt_referencia` |
| RN02 | O cálculo só pode rodar se a validação retornar sucesso | criar o estado `ChecarValidacao`; avançar apenas quando `status_validacao = SUCESSO` | `Glue Job 1.status_validacao` |
| RN03 | A publicação só pode rodar se o cálculo retornar sucesso | criar o estado `ChecarCalculo`; avançar apenas quando `status_calculo = SUCESSO` | `Glue Job 2.status_calculo` |
| RN04 | Qualquer falha em qualquer etapa deve encerrar a execução em falha | direcionar para o estado `Falha` quando qualquer status vier diferente de sucesso ou houver erro técnico do job | `Glue Job 1.status_validacao`, `Glue Job 2.status_calculo`, `Glue Job 3.status_publicacao` |
| RN05 | Execução com publicação concluída deve encerrar em sucesso | direcionar para o estado `Sucesso` somente quando `status_publicacao = SUCESSO` | `Glue Job 3.status_publicacao` |
| RN06 | A competência processada deve ser a mesma em todas as etapas | propagar `dt_referencia` como parâmetro obrigatório na chamada dos 3 jobs | `state_machine_input.dt_referencia` |

## 5. Fluxo de Implementação

1. Ler o payload de entrada da execução da Step Function.
2. Validar a presença de `dt_referencia` no início da execução.
3. Disparar o `Glue Job 1` com `dt_referencia`.
4. Ler `status_validacao`, `quantidade_lida` e `quantidade_invalidos`.
5. Fazer `Choice` para seguir apenas se `status_validacao = SUCESSO`.
6. Disparar o `Glue Job 2` com a mesma `dt_referencia`.
7. Ler `status_calculo`, `quantidade_processada` e `quantidade_rejeitada`.
8. Fazer `Choice` para seguir apenas se `status_calculo = SUCESSO`.
9. Disparar o `Glue Job 3` com a mesma `dt_referencia`.
10. Ler `status_publicacao` e `particao_publicada`.
11. Encerrar em `Sucesso` apenas quando a publicação retornar sucesso.
12. Encerrar em `Falha` para qualquer erro técnico, schema inválido, cálculo inválido ou falha de publicação.

## 6. Saída Esperada

### 6.1 Dataset/tabela/artefato de saída

| Campo | Origem | Regra |
|---|---|---|
| `execution_status` | Step Function | `SUCESSO` ou `FALHA` conforme os `Choice` da orquestração |
| `dt_referencia` | `state_machine_input.dt_referencia` | cópia direta para rastreabilidade da execução |
| `status_validacao` | `Glue Job 1.status_validacao` | retorno do Job 1 |
| `status_calculo` | `Glue Job 2.status_calculo` | retorno do Job 2 |
| `status_publicacao` | `Glue Job 3.status_publicacao` | retorno do Job 3 |
| `particao_publicada` | `Glue Job 3.particao_publicada` | preenchido quando a publicação concluir |

### 6.2 Critérios de aceite técnicos

- a execução deve falhar se `dt_referencia` não for informado
- o cálculo não pode executar quando a validação falhar
- a publicação não pode executar quando o cálculo falhar
- a execução não pode terminar em sucesso sem `status_publicacao = SUCESSO`
- a Step Function deve usar exatamente os estados definidos na RFC: `ValidarEntrada`, `ChecarValidacao`, `CalcularVariaveis`, `ChecarCalculo`, `PublicarTabelaFinal`, `Sucesso` e `Falha`

## 7. Restrições

- não adicionar serviços de orquestração fora de Step Functions
- não inventar novos jobs fora dos 3 Glue Jobs previstos na RFC
- não alterar o contrato lógico de sucesso/falha descrito na RFC
- premissa: os 3 Glue Jobs expõem status lógico compatível com os campos documentados acima

## 8. Entregáveis de código

- definição da state machine da Step Function
- payload padrão de entrada com `dt_referencia`
- integração da Step Function com os 3 Glue Jobs
- tratamento explícito de sucesso e falha por etapa
- testes de fluxo cobrindo caminho feliz e falhas em cada job

## 9. Estimativa

| Bloco | Esforço |
|---|---|
| modelagem do payload e contrato de execução | 0,5 dia |
| definição dos estados, tasks e choices | 0,5 dia |
| integração com os 3 Glue Jobs | 0,5 dia |
| tratamento de falhas e término da execução | 0,25 dia |
| testes de orquestração | 0,25 dia |
| total | 2 dias úteis |

**Tamanho:** `M`
