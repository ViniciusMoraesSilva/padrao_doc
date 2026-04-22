# Prompt de Implementação - H-05 Observabilidade do Pipeline

## 1. Objetivo da História

Implementar a observabilidade operacional do pipeline com CloudWatch e Datadog.
A solução deve centralizar logs dos jobs, emitir métricas mínimas por execução e configurar alarmes para falha, duração, volume e rejeições, conforme a RFC.

## 2. Contexto

Esta história depende de H-01, H-02, H-03 e H-04, porque a observabilidade precisa instrumentar a orquestração e os jobs reais.
Ela habilita H-06 ao fornecer evidências operacionais para a validação final do pipeline.

A RFC define CloudWatch como fonte primária de logs, métricas e alarmes, e Datadog como consolidador da visão operacional.

## 3. Entradas Obrigatórias

### 3.1 Tabelas / arquivos / recursos de origem

| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| `Step Function execution logs` | logs | CloudWatch | rastrear início, fim e falha da execução |
| `Glue Job 1 logs` | logs | CloudWatch | diagnosticar validação da entrada |
| `Glue Job 2 logs` | logs | CloudWatch | diagnosticar cálculo e rejeições |
| `Glue Job 3 logs` | logs | CloudWatch | diagnosticar publicação final |
| `CloudWatch Metrics` | métricas | AWS CloudWatch | consolidar duração, volume e rejeições |
| `Datadog integration` | monitoramento | Datadog | consolidar dashboards e alertas operacionais |

### 3.2 Campos por origem

| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| `Step Function execution logs` | `dt_referencia` | string/date | Sim | correlacionar a execução da competência |
| `Step Function execution logs` | `execution_status` | string | Sim | métrica de sucesso ou falha |
| `CloudWatch Metrics` | `duracao_pipeline` | number | Sim | alarme de SLA mensal |
| `CloudWatch Metrics` | `duracao_job` | number | Sim | acompanhamento por job |
| `CloudWatch Metrics` | `volume_lido` | number | Sim | detectar quebra de carga |
| `CloudWatch Metrics` | `registros_processados` | number | Sim | acompanhar throughput |
| `CloudWatch Metrics` | `registros_rejeitados` | number | Sim | detectar degradação de qualidade |
| `CloudWatch Metrics` | `falhas_execucao` | number | Sim | alarme operacional |

## 4. Regras que devem virar código

| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | Todos os jobs devem enviar logs para CloudWatch | configurar logs estruturados por job e por `dt_referencia` | logs da Step Function e dos 3 Glue Jobs |
| RN02 | O pipeline deve emitir métricas mínimas de duração | publicar `duracao_pipeline` e `duracao_job` por execução | CloudWatch Metrics |
| RN03 | O pipeline deve emitir métricas de volume e rejeição | publicar `volume_lido`, `registros_processados` e `registros_rejeitados` | CloudWatch Metrics |
| RN04 | Falhas do pipeline devem gerar alerta operacional | configurar alarme quando a execução encerrar com falha | `execution_status`, `falhas_execucao` |
| RN05 | Degradação relevante deve ser detectável | configurar alarme para excesso de duração, quebra de carga e pico de rejeição | `duracao_pipeline`, `volume_lido`, `registros_rejeitados` |
| RN06 | O Datadog deve consolidar a visão de saúde do pipeline | expor os sinais do CloudWatch em dashboard/painel operacional | logs e métricas integrados ao Datadog |

## 5. Fluxo de Implementação

1. Instrumentar a Step Function para registrar início, fim e falha com `dt_referencia`.
2. Instrumentar os 3 Glue Jobs para emitir logs estruturados por execução.
3. Publicar métricas de duração por job e duração total do pipeline.
4. Publicar métricas de volume lido, processado e rejeitado.
5. Configurar alarmes no CloudWatch para falha, duração acima do SLA e degradação relevante.
6. Integrar sinais no Datadog para visão consolidada do pipeline.

## 6. Saída Esperada

### 6.1 Dataset/tabela/artefato de saída

| Campo | Origem | Regra |
|---|---|---|
| `dt_referencia` | logs da Step Function e jobs | cópia direta para correlação |
| `execution_status` | derivado da execução | sucesso ou falha da execução |
| `duracao_pipeline` | derivado | métrica agregada da execução completa |
| `duracao_job` | derivado | métrica por job executado |
| `volume_lido` | derivado do Job 1 | quantidade lida da origem |
| `registros_processados` | derivado do Job 2/3 | quantidade efetivamente processada |
| `registros_rejeitados` | derivado do Job 1/2 | quantidade rejeitada por regra estrutural ou de cálculo |

### 6.2 Critérios de aceite técnicos

- todos os jobs devem ter logs acessíveis no CloudWatch
- a competência deve ser identificável nos logs e métricas
- deve existir alarme para falha do pipeline
- deve existir alarme para duração acima do SLA mensal
- deve existir visão consolidada no Datadog

## 7. Restrições

- não alterar lógica funcional dos jobs para além da instrumentação necessária
- não remover logs ou métricas mínimas exigidas pela RFC
- não registrar dados sensíveis além do estritamente necessário para operação
- premissa: o detalhamento de thresholds de alarmes de volume e rejeição será ajustado com base nas primeiras execuções, pois a RFC não define valores numéricos exatos

## 8. Entregáveis de código

- instrumentação de logs da Step Function e dos 3 Glue Jobs
- emissão de métricas customizadas no CloudWatch
- configuração de alarmes operacionais mínimos
- integração/painel operacional no Datadog
- testes operacionais de emissão de logs, métricas e disparo de alertas

## 9. Estimativa

| Bloco | Esforço |
|---|---|
| definição dos sinais obrigatórios | 0,5 dia |
| instrumentação dos jobs e orquestração | 0,5 dia |
| configuração de métricas e alarmes | 0,5 dia |
| integração com Datadog | 0,25 dia |
| testes operacionais | 0,25 dia |
| total | 2 dias úteis |

**Tamanho:** `M`
