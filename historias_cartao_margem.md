# Histórias — Pipeline de Margem de Rentabilidade — Cartão Alfa

**Origem:** prd_cartao_margem.md / tech_spec_cartao_margem.md
**Responsável pela criação no board:** Ana Beatriz Torres (PO)
**Data:** 28/04/2026

---

## H-01 — Job 1: Ingestão das origens para staging S3

**Título:** Criar Job Spark de ingestão dos dados das 3 origens para a staging area no S3

Como pipeline de margem do Cartão Alfa,
quero coletar e consolidar os dados de transações (S3), saldo/limite/score (CORE_BANK) e faixas de PDD (RISK_ENGINE)
para disponibilizá-los em staging no S3 prontos para o cálculo.

**Contexto:**
Primeiro job do pipeline. Toda a lógica de cálculo das histórias seguintes depende desta staging estar correta e completa. Falha em qualquer origem deve interromper o pipeline e notificar o time — não deve avançar com dado parcial.

**Critérios de aceite**
- CA1: Lê transações do S3 (`transacoes_cartao/`) filtrando apenas `cd_tipo_transacao = 'COMPRA'` dos últimos 12 meses e agrega a média mensal de `vl_mdr` por CPF
- CA2: Lê `vl_saldo_devedor`, `vl_limite_aprovado` e `nr_score_credito` do CORE_BANK via JDBC com leitura particionada por faixa de CPF
- CA3: Lê o arquivo `pdd_faixas_vigente.parquet` do S3 e valida que `dt_vigencia` tem no máximo 2 dias de defasagem — falha com erro descritivo se estiver desatualizado
- CA4: Persiste os 3 conjuntos de dados em `s3://cartao-margem/staging/{ano}/{mes}/` em formato Parquet
- CA5: Falha em qualquer origem encerra o job com erro, registra no CloudWatch qual sistema falhou e não grava dados parciais na staging
- CA6: Job é idempotente — reexecutar para a mesma partição de data sobrescreve sem duplicar

**Dependências**
- Acesso JDBC ao CORE_BANK liberado pela Infra
- Permissão IAM de leitura no bucket `s3://datalake-prod/` e `s3://risk-exports/`
- Permissão IAM de escrita no bucket `s3://cartao-margem/`

**Sizing:** M

**Tasks sugeridas**
- Configurar conexão Spark JDBC ao CORE_BANK com `numPartitions` por faixa de CPF
- Implementar leitura do S3 de transações com filtro de data e tipo
- Implementar validação de `dt_vigencia` do arquivo de PDD
- Implementar lógica de falha controlada (sem dados parciais)
- Escrever testes com dados de amostra em homologação (1.000 clientes)

---

## H-02 — Job 2: Cálculo de receitas e custos (RN01 a RN04)

**Título:** Criar Job Spark de cálculo dos componentes de receita e custo da margem por cliente

Como pipeline de margem do Cartão Alfa,
quero calcular receita de intercâmbio, receita de juros, custo de PDD e custo operacional por cliente
para ter os componentes individuais da margem prontos para o cálculo final.

**Contexto:**
Segundo job do pipeline. Lê da staging gerada pelo H-01, aplica as 4 primeiras regras de negócio e persiste o resultado intermediário em S3. Os parâmetros variáveis (custo operacional, taxa de juros) são lidos de um arquivo JSON em S3 — sem necessidade de redeploy para ajuste.

**Critérios de aceite**
- CA1: **RN01** — `vl_receita_intercambio` = média mensal de `vl_mdr` das transações de compra dos últimos 12 meses (já agregado no Job 1)
- CA2: **RN02** — `vl_receita_juros` = `vl_saldo_devedor_medio × taxa_juros_rotativo`; deve ser `0` quando `vl_saldo_devedor_medio = 0`; taxa lida do arquivo de parâmetros em S3
- CA3: **RN03** — `vl_custo_pdd` = `vl_limite_aprovado × pc_pdd` onde `pc_pdd` é obtido pelo join com a faixa de score do cliente; clientes com `nr_score_credito < 500` (sem faixa válida) são excluídos e registrados em log de exclusão com CPF mascarado
- CA4: **RN04** — `vl_custo_operacional` = valor lido do campo `vl_custo_operacional` do arquivo `s3://cartao-margem/params/params.json`; atualmente R$ 9,20
- CA5: Resultado persistido em `s3://cartao-margem/calculado/{ano}/{mes}/` com todos os componentes individuais por coluna (não só o total)
- CA6: Job lê parâmetros do arquivo S3 no início da execução — qualquer alteração no arquivo é refletida no próximo ciclo sem redeploy

**Dependências:** H-01 concluída e staging disponível em S3

**Sizing:** M

**Tasks sugeridas**
- Implementar join da staging de CORE_BANK com o Parquet de faixas de PDD por faixa de score
- Implementar leitura do `params.json` no início do job
- Implementar RN01, RN02, RN03 e RN04 como colunas separadas no DataFrame
- Implementar exclusão de clientes com score < 500 com log de CPF mascarado
- Testes unitários: cliente com saldo zero (RN02 = 0), cliente em cada faixa de score (RN03), cliente sem faixa válida (exclusão)

---

## H-03 — Job 3: Margem líquida, segmentação e saída

**Título:** Criar Job Spark de cálculo da margem líquida, segmentação e geração das saídas (Redshift + CSV S3)

Como pipeline de margem do Cartão Alfa,
quero calcular a margem líquida final por cliente, classificar em PREMIUM / PADRÃO / RISCO e gerar as saídas para o time comercial e para auditoria
para que o rollout possa ser executado com base nos segmentos corretos.

**Contexto:**
Terceiro e último job de processamento. Lê o resultado do H-02, aplica RN05 e gera duas saídas: persistência histórica no Redshift (com todos os componentes para auditoria) e arquivo CSV de rollout em S3 (com CPF mascarado, apenas segmentos PREMIUM e PADRÃO).

**Critérios de aceite**
- CA1: **RN05** — `vl_margem_projetada = vl_receita_total − vl_custo_total` onde `vl_receita_total = RN01 + RN02` e `vl_custo_total = RN03 + RN04`
- CA2: Segmentação: PREMIUM se `vl_margem_projetada ≥ 60,00` / PADRÃO se `20,00 ≤ vl_margem_projetada < 60,00` / RISCO se `vl_margem_projetada < 20,00`
- CA3: Resultado persistido na tabela `tb_margem_cliente` no Redshift em modo `append`, incluindo colunas individuais de cada componente (RN01 a RN04)
- CA4: Arquivo CSV gerado em `s3://cartao-margem/rollout/{ano}/{mes}/rollout_segmentado.csv` contendo apenas clientes PREMIUM e PADRÃO, com CPF no formato `XXX*****XX` (primeiros 3 + últimos 2 dígitos)
- CA5: CPF nunca aparece sem mascaramento em nenhum log, arquivo intermediário ou arquivo de saída
- CA6: Reprocessamento do job para a mesma partição de data faz `upsert` no Redshift (não duplica) e sobrescreve o CSV em S3

**Dependências:** H-02 concluída e resultado disponível em `s3://cartao-margem/calculado/`

**Sizing:** M

**Tasks sugeridas**
- Implementar RN05 (margem e segmentação) com testes de borda: margem exatamente em R$ 20,00 e R$ 60,00
- Implementar escrita no Redshift via Spark JDBC com lógica de `upsert` por `(nr_cpf, dt_calculo)`
- Implementar mascaramento de CPF antes de qualquer persistência ou log
- Gerar e persistir o CSV de rollout em S3 com header
- Testes com amostra real em homologação: validar contagem por segmento contra expectativa do time de Risco

---

## H-04 — Orquestração Step Functions, parâmetros e notificação SNS

**Título:** Configurar orquestração Step Functions com encadeamento dos jobs, arquivo de parâmetros e notificação SNS

Como squad de Cartões,
quero que o pipeline rode automaticamente todo dia 1 do mês, encadeie os 3 jobs em sequência e notifique os times certos ao final (sucesso ou falha)
para que o processo seja confiável e não dependa de execução manual.

**Contexto:**
A Step Function orquestra os 3 jobs EMR Serverless em sequência. Qualquer falha interrompe o fluxo e notifica o SRE via SNS. Ao sucesso do Job 3, notifica o time comercial com o link do arquivo de rollout. O arquivo de parâmetros (`params.json`) é gerenciado pela área de Finanças diretamente no S3 — sem necessidade de deploy.

**Critérios de aceite**
- CA1: Step Function encadeia Job1 → Job2 → Job3 em sequência; falha em qualquer job interrompe o fluxo e não executa os próximos
- CA2: Em caso de falha em qualquer job, SNS notifica o canal de SRE com identificação do job que falhou
- CA3: Ao término bem-sucedido do Job 3, SNS notifica o time comercial com o path do arquivo de rollout gerado
- CA4: Execução agendada via EventBridge no dia 1 de cada mês às 00h00 (horário de Brasília)
- CA5: Arquivo `s3://cartao-margem/params/params.json` estruturado com os campos `vl_custo_operacional`, `vl_limite_premium`, `vl_limite_padrao` e `taxa_juros_rotativo` — alteração do arquivo é refletida na próxima execução sem redeploy

**Dependências:** H-01, H-02 e H-03 concluídas e funcionando em homologação

**Sizing:** P

**Tasks sugeridas**
- Criar definição da Step Function com estados para Job1, Job2, Job3, Falha_Notificar e Notificar_Comercial
- Configurar trigger EventBridge (dia 1 do mês, 00h00)
- Configurar tópicos SNS para SRE e time comercial
- Criar e versionar `params.json` no S3 com valores iniciais validados por Finanças e Risco
- Testar execução completa em homologação (execução manual da Step Function com dados de amostra)

---

## H-05 — Observabilidade: métricas, logs e alertas no CloudWatch

**Título:** Implementar observabilidade do pipeline com métricas, logs estruturados e alertas no CloudWatch

Como squad e SRE,
quero visibilidade sobre o funcionamento do pipeline a cada execução
para identificar falhas, anomalias de dado e problemas de performance rapidamente.

**Contexto:**
O pipeline roda durante a madrugada do dia 1. Sem observabilidade, qualquer falha só seria percebida quando o time comercial tentasse usar o arquivo de rollout. Os alertas devem cobrir tanto falha técnica (job parou) quanto anomalia de negócio (% de PREMIUM fora do esperado).

**Critérios de aceite**
- CA1: Dashboard CloudWatch com: duração total do pipeline, total de clientes processados, contagem por segmento (PREMIUM / PADRÃO / RISCO), clientes excluídos por score < 500
- CA2: Alarme P1 (PagerDuty) disparado se o pipeline não concluir até 06h00 do dia 1
- CA3: Alarme de anomalia de negócio disparado se % de clientes PREMIUM for < 5% ou > 70% após o Job 3 — notificação para PO (Ana Torres) e Risco (Rafael Gomes)
- CA4: Alarme disparado antes do início do pipeline se `pdd_faixas_vigente.parquet` tiver `dt_vigencia` com mais de 2 dias — notificação para o time de Risco
- CA5: Nenhum log emite CPF sem mascaramento em nenhum dos 3 jobs

**Dependências:** H-04 concluída

**Sizing:** P

**Tasks sugeridas**
- Configurar métricas customizadas no CloudWatch emitidas ao fim de cada job (contagem de registros, duração, erros)
- Criar alarme de duração total > 6h (P1 → PagerDuty)
- Criar alarme de anomalia de segmento (% PREMIUM fora da faixa)
- Criar verificação de `dt_vigencia` do PDD como pré-condição do Job 1 com alarme SNS
- Validar mascaramento de CPF nos logs dos 3 jobs antes de considerar pronto
