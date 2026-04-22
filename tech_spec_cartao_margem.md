# Tech Spec / RFC — Pipeline de Margem de Rentabilidade — Cartão Alfa

**PRD de origem:** prd_cartao_margem.md
**Tech Lead:** Bruno Ferreira · **Revisores:** Carla Dias (Dados), Paulo Vasconcelos (SRE)
**Data:** 28/04/2026 · **Status:** Em review · **Versão:** v1

---

## 1. Resumo técnico (TL;DR)

Pipeline batch mensal orquestrado por AWS Step Functions com 3 jobs Spark (EMR Serverless). O Job 1 ingere dados de 3 origens para uma staging area no S3. O Job 2 aplica as 5 regras de negócio e calcula margem por cliente. O Job 3 segmenta os clientes, persiste o resultado no Redshift e gera um arquivo CSV por segmento no S3. Ao final, um SNS notifica o time comercial. Não há exposição via API — todo o fluxo é orientado a arquivo e banco analítico.

---

## 2. Contexto técnico atual

Não existe pipeline de rentabilidade para o Cartão Alfa. A elegibilidade é feita direto no motor de ofertas consultando CORE_BANK e RISK_ENGINE via JDBC/API em tempo real. Não há dados de margem armazenados em nenhuma camada analítica.

Infraestrutura disponível que será reutilizada:
- AWS EMR Serverless (já aprovisionado para outros pipelines da squad)
- S3 como Datalake (partições Parquet já usadas por outros produtos)
- Redshift como DW analítico (acesso via JDBC)
- Step Functions para orquestração de pipelines batch
- SNS para notificações operacionais

---

## 3. Insumos recebidos do PRD

| Item | Status |
|---|---|
| Tabelas de origem identificadas no catálogo Blue | Confirmado |
| Campos necessários por tabela mapeados pelo PO | Confirmado |
| 5 regras de negócio (RN01 a RN05) com responsável de validação | Confirmado |
| Faixas de PDD por score (RN03) — tabela `pdd_faixas_score` | Confirmado com Rafael Gomes |
| Custo operacional fixo R$ 9,20 (RN04) | Confirmado com Camila Neves |
| Tabela `fatura_cartao` tem histórico dos últimos 12 meses ou só última fatura | **Pendente** — Core Banking (prazo 25/04) |
| Campo `vl_mdr` disponível para todos os clientes em `transacoes_cartao` | **Pendente** — Carla Dias (prazo 25/04) |

> Lacunas assumidas nesta spec: `fatura_cartao` tem histórico dos últimos 12 meses disponível; `transacoes_cartao` tem `vl_mdr` para todos os clientes; saldo devedor calculado como média dos últimos 12 meses.

---

## 4. Arquitetura da solução

```
Dia 1 do mês (00h00)
        │
        ▼
┌──────────────────────────────────────────────┐
│           AWS Step Functions                 │
│                                              │
│  ┌─────────────────────────────────────────┐ │
│  │  Job 1 — Ingestão (Spark / EMR)         │ │
│  │                                         │ │
│  │  Origem 1: S3 Datalake (Parquet)        │ │
│  │    transacoes_cartao/ano=*/mes=*/       │ │
│  │    → filtra 12 meses, cd_tipo='COMPRA'  │ │
│  │                                         │ │
│  │  Origem 2: Core Banking (JDBC)           │ │
│  │    fatura_cartao + limite_credito       │ │
│  │    → saldo devedor médio, limite, score │ │
│  │                                         │ │
│  │  Origem 3: Risco de Crédito (catálogo)  │ │
│  │    pdd_faixas_score                     │ │
│  │    → faixas de score e % de PDD         │ │
│  │                                         │ │
│  │  Saída: s3://cartao-margem/staging/     │ │
│  └────────────────────┬────────────────────┘ │
│                       │ sucesso              │
│  ┌────────────────────▼────────────────────┐ │
│  │  Job 2 — Cálculo de Margem (Spark)      │ │
│  │                                         │ │
│  │  Lê da staging, aplica RN01 a RN05      │ │
│  │  por cliente                            │ │
│  │                                         │ │
│  │  Saída: s3://cartao-margem/calculado/   │ │
│  └────────────────────┬────────────────────┘ │
│                       │ sucesso              │
│  ┌────────────────────▼────────────────────┐ │
│  │  Job 3 — Segmentação e Saída (Spark)    │ │
│  │                                         │ │
│  │  Aplica RN05 (classificação)            │ │
│  │  Persiste em Redshift (tb_margem)       │ │
│  │  Gera CSV por segmento em S3            │ │
│  │                                         │ │
│  │  Saída: Redshift + S3 rollout CSV       │ │
│  └────────────────────┬────────────────────┘ │
│                       │ sucesso              │
│  ┌────────────────────▼────────────────────┐ │
│  │  SNS → notificação ao time comercial    │ │
│  └─────────────────────────────────────────┘ │
│                                              │
│  Em qualquer falha: Step Functions para      │
│  no job com erro, registra no CloudWatch     │
│  e dispara alerta SNS para o time de SRE    │
└──────────────────────────────────────────────┘
```

---

## 5. Detalhamento dos Jobs Spark

### Job 1 — Ingestão

**Entrada:** 3 origens (S3 Parquet, JDBC, S3 Parquet)
**Saída:** `s3://cartao-margem/staging/ano={ano}/mes={mes}/`

Lógica:

```python
# Leitura das transações (Origem 1)
df_txn = (
    spark.read.parquet("s3://datalake-prod/transacoes_cartao/")
    .filter(col("cd_tipo_transacao") == "COMPRA")
    .filter(col("dt_transacao") >= data_inicio_12_meses)
)

# Agregação: média mensal de MDR por CPF
df_intercambio = (
    df_txn
    .groupBy("nr_cpf", year("dt_transacao").alias("ano"), month("dt_transacao").alias("mes"))
    .agg(sum("vl_mdr").alias("vl_mdr_mes"))
    .groupBy("nr_cpf")
    .agg(avg("vl_mdr_mes").alias("vl_receita_intercambio"))
)

# Leitura do CORE_BANK (Origem 2)
df_core = (
    spark.read
    .format("jdbc")
    .option("url", JDBC_URL_CORE_BANK)
    .option("dbtable", """
        SELECT f.nr_cpf,
               AVG(f.vl_saldo_devedor) AS vl_saldo_devedor_medio,
               l.vl_limite_aprovado,
               c.nr_score_credito
        FROM   fatura_cartao f
        JOIN   limite_credito l ON l.nr_cpf = f.nr_cpf
        JOIN   cliente_cartao c ON c.nr_cpf = f.nr_cpf
        WHERE  f.dt_referencia >= ADD_MONTHS(TRUNC(SYSDATE,'MM'), -12)
        GROUP  BY f.nr_cpf, l.vl_limite_aprovado, c.nr_score_credito
    """)
    .load()
)

# Leitura das faixas de PDD (Origem 3 — tabela pdd_faixas_score do catálogo Blue)
df_pdd = spark.read.table("pdd_faixas_score")

# Persistência na staging
df_intercambio.write.mode("overwrite").parquet(f"s3://cartao-margem/staging/{particao}/intercambio/")
df_core.write.mode("overwrite").parquet(f"s3://cartao-margem/staging/{particao}/core/")
df_pdd.write.mode("overwrite").parquet(f"s3://cartao-margem/staging/{particao}/pdd/")
```

---

### Job 2 — Cálculo de Margem

**Entrada:** `s3://cartao-margem/staging/{particao}/`
**Saída:** `s3://cartao-margem/calculado/{particao}/`

Regras aplicadas:

| Regra | Cálculo |
|---|---|
| RN01 — Receita de intercâmbio | `vl_receita_intercambio` (já calculado no Job 1) |
| RN02 — Receita de juros | `vl_saldo_devedor_medio × 0.125` (somente se `vl_saldo_devedor_medio > 0`) |
| RN03 — Custo de PDD | `vl_limite_aprovado × pc_pdd` (join com faixa de score do cliente) |
| RN04 — Custo operacional | `9.20` (constante lida do arquivo de parâmetros em S3) |
| RN05 — Margem líquida | `(RN01 + RN02) − (RN03 + RN04)` |

Lógica:

```python
# Join das tabelas de staging
df = (
    df_intercambio
    .join(df_core, "nr_cpf", "inner")
    .join(df_pdd,
          (df_core["nr_score_credito"] >= df_pdd["score_min"]) &
          (df_core["nr_score_credito"] <= df_pdd["score_max"]),
          "left")
)

# Exclusão de clientes com score < 500 (sem faixa de PDD válida)
df = df.filter(col("pc_pdd").isNotNull())

# Parâmetros lidos de S3 (sem necessidade de redeploy)
params = spark.read.json("s3://cartao-margem/params/params.json").first()
custo_operacional = params["vl_custo_operacional"]  # 9.20

# Cálculo das regras
df = df.withColumn(
    "vl_receita_juros",
    when(col("vl_saldo_devedor_medio") > 0, col("vl_saldo_devedor_medio") * 0.125).otherwise(0)
).withColumn(
    "vl_custo_pdd",
    col("vl_limite_aprovado") * col("pc_pdd")
).withColumn(
    "vl_custo_operacional", lit(custo_operacional)
).withColumn(
    "vl_receita_total", col("vl_receita_intercambio") + col("vl_receita_juros")
).withColumn(
    "vl_custo_total", col("vl_custo_pdd") + col("vl_custo_operacional")
).withColumn(
    "vl_margem_projetada", col("vl_receita_total") - col("vl_custo_total")
)

df.write.mode("overwrite").parquet(f"s3://cartao-margem/calculado/{particao}/")
```

---

### Job 3 — Segmentação e Saída

**Entrada:** `s3://cartao-margem/calculado/{particao}/`
**Saída:** Redshift (`tb_margem_cliente`) + CSV em S3 por segmento

```python
# Segmentação (RN05)
# Mapeamento para nomenclatura do PRD:
#   PREMIUM = Alta Margem | PADRAO = Margem Média | RISCO = Margem Baixa
df = df.withColumn(
    "cd_segmento_margem",
    when(col("vl_margem_projetada") >= 60.0, "PREMIUM")
    .when(col("vl_margem_projetada") >= 20.0, "PADRAO")
    .otherwise("RISCO")
).withColumn("dt_calculo", lit(data_referencia))

# Persistência no Redshift (auditoria histórica)
(
    df.select(
        "nr_cpf", "dt_calculo", "cd_segmento_margem",
        "vl_margem_projetada", "vl_receita_total", "vl_custo_total",
        "vl_receita_intercambio", "vl_receita_juros",
        "vl_custo_pdd", "vl_custo_operacional"
    )
    .write
    .format("jdbc")
    .option("url", JDBC_URL_REDSHIFT)
    .option("dbtable", "tb_margem_cliente")
    .mode("append")
    .save()
)

# Geração do CSV de rollout por segmento (CPF mascarado)
df_rollout = (
    df.filter(col("cd_segmento_margem").isin(["PREMIUM", "PADRAO"]))
    .withColumn(
        "nr_cpf_mascarado",
        concat(substring("nr_cpf", 1, 3), lit("*****"), substring("nr_cpf", 9, 2))
    )
    .select("nr_cpf_mascarado", "cd_segmento_margem", "vl_margem_projetada", "dt_calculo")
)

df_rollout.write.mode("overwrite").csv(
    f"s3://cartao-margem/rollout/{ano}/{mes}/rollout_segmentado.csv",
    header=True
)
```

---

## 6. Schema do Redshift

```sql
CREATE TABLE tb_margem_cliente (
    nr_cpf                  VARCHAR(11)    NOT NULL,
    dt_calculo              DATE           NOT NULL,
    cd_segmento_margem      VARCHAR(10)    NOT NULL,  -- PREMIUM | PADRAO | RISCO
    vl_margem_projetada     NUMERIC(12,2)  NOT NULL,
    vl_receita_total        NUMERIC(12,2)  NOT NULL,
    vl_custo_total          NUMERIC(12,2)  NOT NULL,
    vl_receita_intercambio  NUMERIC(12,2),
    vl_receita_juros        NUMERIC(12,2),
    vl_custo_pdd            NUMERIC(12,2),
    vl_custo_operacional    NUMERIC(12,2),
    dt_processamento        TIMESTAMP      DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (nr_cpf, dt_calculo)
)
DISTKEY(nr_cpf)
SORTKEY(dt_calculo);
```

---

## 7. Arquivo de parâmetros (sem redeploy)

```json
// s3://cartao-margem/params/params.json
{
  "vl_custo_operacional": 9.20,
  "vl_limite_premium": 60.0,
  "vl_limite_padrao": 20.0,
  "taxa_juros_rotativo": 0.125
}
```

> Qualquer alteração de parâmetro de negócio (ex: custo operacional ajustado por Finanças) é feita atualizando esse arquivo em S3 — sem necessidade de deploy ou alteração de código.

---

## 8. Orquestração com Step Functions

```json
{
  "Comment": "Pipeline de Margem Cartão Alfa",
  "StartAt": "Job1_Ingestao",
  "States": {
    "Job1_Ingestao": {
      "Type": "Task",
      "Resource": "arn:aws:states:::emr-serverless:startJobRun.sync",
      "Parameters": {
        "ApplicationId": "${EMR_APP_ID}",
        "ExecutionRoleArn": "${EMR_ROLE_ARN}",
        "JobDriver": {
          "SparkSubmit": {
            "EntryPoint": "s3://cartao-margem/jobs/job1_ingestao.py",
            "SparkSubmitParameters": "--conf spark.executor.memory=8g"
          }
        }
      },
      "Next": "Job2_Calculo",
      "Catch": [{ "ErrorEquals": ["States.ALL"], "Next": "Falha_Notificar" }]
    },
    "Job2_Calculo": {
      "Type": "Task",
      "Resource": "arn:aws:states:::emr-serverless:startJobRun.sync",
      "Parameters": {
        "ApplicationId": "${EMR_APP_ID}",
        "ExecutionRoleArn": "${EMR_ROLE_ARN}",
        "JobDriver": {
          "SparkSubmit": {
            "EntryPoint": "s3://cartao-margem/jobs/job2_calculo.py"
          }
        }
      },
      "Next": "Job3_Saida",
      "Catch": [{ "ErrorEquals": ["States.ALL"], "Next": "Falha_Notificar" }]
    },
    "Job3_Saida": {
      "Type": "Task",
      "Resource": "arn:aws:states:::emr-serverless:startJobRun.sync",
      "Parameters": {
        "ApplicationId": "${EMR_APP_ID}",
        "ExecutionRoleArn": "${EMR_ROLE_ARN}",
        "JobDriver": {
          "SparkSubmit": {
            "EntryPoint": "s3://cartao-margem/jobs/job3_saida.py"
          }
        }
      },
      "Next": "Notificar_Comercial",
      "Catch": [{ "ErrorEquals": ["States.ALL"], "Next": "Falha_Notificar" }]
    },
    "Notificar_Comercial": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "${SNS_COMERCIAL_ARN}",
        "Message": "Pipeline de margem concluído. Arquivo disponível em s3://cartao-margem/rollout/"
      },
      "End": true
    },
    "Falha_Notificar": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "${SNS_SRE_ARN}",
        "Message": "FALHA no pipeline de margem Cartão Alfa. Verificar CloudWatch."
      },
      "End": true
    }
  }
}
```

---

## 9. Alternativas consideradas

| Alternativa | Status | Motivo |
|---|---|---|
| Orquestrar com Airflow (MWAA) | Descartada | Step Functions já aprovisionado para pipelines da squad; menos operação |
| Processar em Lambda (sem Spark) | Descartada | Volume de ~480k clientes com join de 12 meses de transações exige processamento distribuído |
| Expor resultado via API REST | Descartada | Escopo do PRD é exclusivamente batch; API é escopo futuro |
| dbt para o cálculo | Descartada | CORE_BANK (Oracle) não tem conector dbt aprovado no ambiente; Spark JDBC é o caminho padrão |

---

## 10. Requisitos não-funcionais

- **Performance:** pipeline completo (3 jobs) deve finalizar em até 4 horas (janela 00h–06h do dia 1)
- **Escalabilidade:** EMR Serverless escala automaticamente; sem necessidade de pré-configurar cluster
- **Segurança:** CPF nunca aparece em logs ou arquivos de saída sem mascaramento; acesso ao S3 e Redshift via IAM Roles com princípio de menor privilégio
- **Auditoria:** cada linha do Redshift registra os valores componentes individuais (receitas e custos) por regra, não só o total

---

## 11. Estratégia de rollout e rollback

- **Fase 1:** rodar pipeline em paralelo com o fluxo atual (shadow mode) por 1 ciclo — comparar resultados sem impactar oferta
- **Fase 2:** habilitar uso do segmento Alta Margem (PREMIUM) no motor de ofertas (configuração externa, sem deploy)
- **Fase 3:** habilitar Margem Média (PADRAO) após validação da Fase 2
- **Rollback:** desabilitar uso do segmento no motor de ofertas reverte ao comportamento atual. Os dados no banco analítico e nos arquivos de saída são preservados para auditoria.

---

## 12. Riscos técnicos

| Risco | Severidade | Mitigação |
|---|---|---|
| JDBC no CORE_BANK (Oracle legado) pode ter lentidão com 480k clientes | Alta | Particionar leitura por faixa de CPF (`numPartitions` no Spark JDBC) e usar view materializada se disponível |
| Tabela `pdd_faixas_score` desatualizada no dia 1 (`dt_vigencia` > 2 dias) | Média | Verificar `dt_vigencia` no Job 1; falhar e alertar se dado tiver mais de 2 dias |
| Particionamento do S3 sem CPF força full scan em 12 partições de data | Média | Usar pushdown de filtro por data no Spark; aceitar custo de I/O — confirmar volume com Carla Dias |
| Pipeline falha no Job 3 após Jobs 1 e 2 bem-sucedidos | Baixa | Step Functions para no job com falha; Jobs 1 e 2 são idempotentes — reprocessar só o Job 3 manualmente |

---

## 13. Plano de observabilidade

- **Métricas (CloudWatch):**
  - Duração total do pipeline (alarme se > 4h)
  - Total de clientes processados por segmento após Job 3
  - Clientes excluídos por score < 500 (log de exclusão)
  - Falhas por job (alarme imediato via SNS → SRE)

- **Logs:**
  - Início e fim de cada job com duração e contagem de registros
  - Falhas de leitura de origem com identificação do sistema
  - CPF sempre mascarado nos logs

- **Alertas:**
  - Pipeline não concluído até 06h do dia 1 → PagerDuty P1
  - % de clientes Alta Margem (PREMIUM) < 5% ou > 70% após processamento → alerta PO + Risco (possível erro de regra ou de dado)
  - Tabela `pdd_faixas_score` com `dt_vigencia` de mais de 2 dias → alerta antes de iniciar o pipeline

---

## 14. Proposta de quebra em histórias

| História | Tipo | Dependência |
|---|---|---|
| H-01: Job 1 — Ingestão das tabelas do catálogo Blue para staging | Dados | — |
| H-02: Job 2 — Cálculo de receitas e custos (RN01 a RN04) | Backend / Dados | H-01 concluída |
| H-03: Job 3 — Margem líquida, segmentação e saída (RN05 + Redshift + CSV) | Backend / Dados | H-02 concluída |
| H-04: Orquestração Step Functions + parâmetros S3 + notificação SNS | Infra | H-01, H-02, H-03 concluídas |
| H-05: Observabilidade — CloudWatch métricas, logs e alertas | Infra / Obs. | H-04 concluída |

---

## 15. Estimativa de esforço

| Área | Complexidade | Observação |
|---|---|---|
| Dados / Spark (Jobs 1, 2, 3) | L (10 dias) | 3 origens distintas + lógica de cálculo por regra |
| Infra / Step Functions + SNS | M (3 dias) | Reutiliza infraestrutura existente da squad |
| Infra / Observabilidade | M (3 dias) | CloudWatch + alarmes |
| QA / testes | M (4 dias) | Testes de regra + integração + batch em homologação |

---

## 16. Perguntas em aberto

- [ ] A tabela `fatura_cartao` tem histórico dos últimos 12 meses ou só a fatura mais recente? — responsável: Core Banking — prazo: 25/04/2026
- [ ] O campo `vl_mdr` está disponível para todos os clientes na tabela `transacoes_cartao`? — responsável: Carla Dias — prazo: 25/04/2026

---

## Gate de refinamento — TL + time + PM/PO

- [x] Time entende a solução sem depender exclusivamente do TL?
- [x] Alternativas descartadas documentadas com justificativa?
- [x] Riscos com mitigação definida?
- [x] Ordem de execução dos jobs e dependências claras?
- [x] Estratégia de rollback existe?
- [x] Proposta de histórias é quebrável em itens de sprint?
- [x] Plano de observabilidade definido antes de codar?
- [x] PM/PO tem insumo suficiente para criar as histórias no board?
