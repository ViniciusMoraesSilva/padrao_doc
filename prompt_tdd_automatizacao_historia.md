# Prompt-TDD para Automatização de História

## Objetivo

Este documento transforma o padrão de TDD em um formato mais direto, pensado para ser usado como prompt de implementação por um agente de código. A ideia é reduzir ambiguidade e fazer a história sair do nível “documentação” para o nível “instrução executável”.

O foco é:

- deixar claro o contexto da história
- explicitar quais tabelas precisam ser lidas
- listar os campos obrigatórios e suas origens
- separar regras de negócio implementáveis
- definir a saída esperada
- informar dependências, restrições e estimativa

---

## Padrão de Mercado Incorporado

Este modelo foi adaptado para ficar próximo do que há de mais forte em documentação técnica operacional:

### Google

- contexto e problema antes da solução
- escopo explícito e decisões claras
- documentação curta, objetiva e próxima do código
- contratos e premissas explícitas para reduzir interpretação ambígua

### Microsoft

- separação entre requisito, design técnico, integração e modelo de dados
- uso de documento por processo/história para facilitar ownership
- mapeamento de interfaces e dados no nível de tabela/campo

### Meta

- forte ênfase em schematization
- explicitação de lineage de dados: o que entra, de onde vem, para onde vai
- padronização de contratos de entrada e saída para permitir automação

### Princípio consolidado deste modelo

Se uma regra não puder ser vinculada a:

1. uma tabela de origem
2. um conjunto de campos
3. uma transformação objetiva
4. uma saída verificável

então ela ainda não está pronta para automação.

---

## Estrutura Recomendada de Prompt-TDD

Use este bloco como template base:

```md
# Prompt de Implementação - [ID da História] [Título]

## 1. Objetivo da História
- Descreva em 2 a 4 linhas o que precisa ser implementado.

## 2. Contexto
- Explique o papel da história dentro do fluxo maior.
- Liste a dependência da história anterior e o que esta história habilita na próxima.

## 3. Entradas Obrigatórias

### 3.1 Tabelas / arquivos de origem
| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| tabela_x | tabela | banco/schema | usada para ... |
| arquivo_y | arquivo | s3://... | usado para ... |

### 3.2 Campos por origem
| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| tabela_x | campo_a | decimal | Sim | cálculo da regra RN01 |
| tabela_x | campo_b | string | Sim | chave de join |

## 4. Regras que devem virar código
| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | ... | criar coluna `...` usando `...` | tabela_x.campo_a |

## 5. Fluxo de Implementação
1. Ler tabela/arquivo X
2. Filtrar registros conforme regra Y
3. Fazer join com tabela Z usando chave K
4. Calcular colunas derivadas
5. Persistir saída em local D

## 6. Saída Esperada

### 6.1 Dataset/tabela de saída
| Campo | Origem | Regra |
|---|---|---|
| campo_saida_1 | cópia de tabela_x.campo_a | cópia direta |
| campo_saida_2 | derivado | RN01 |

### 6.2 Critérios de aceite técnicos
- não duplicar registros na reexecução
- falhar se tabela de origem não existir
- falhar se campo obrigatório estiver ausente

## 7. Restrições
- não inventar campos
- não criar regra fora da RFC/PRD
- se houver lacuna, registrar como premissa explícita

## 8. Entregáveis de código
- job/processo/serviço a ser criado
- testes unitários
- testes de integração
- ajuste de observabilidade, se aplicável

## 9. Estimativa
| Bloco | Esforço |
|---|---|
| leitura e joins | 1 dia |
| regras de negócio | 1,5 dia |
| testes | 1 dia |
| total | 3,5 dias |
```

---

## Exemplo Pronto para Uso

Abaixo está um exemplo já preenchido, pensado para ser usado como prompt de automação de código para a história de cálculo de margem do Cartão Alfa.

---

# Prompt de Implementação - H-02 Job 2: Cálculo de Receitas e Custos

## 1. Objetivo da História

Implementar o Job 2 do pipeline batch mensal do Cartão Alfa. Este job deve ler os dados preparados na staging do Job 1, aplicar as regras RN01 a RN04, calcular os componentes financeiros por cliente e persistir o resultado intermediário em S3 para consumo do Job 3.

O código deve ser orientado a dados, idempotente por partição e aderente às regras da RFC e do Tech Spec.

## 2. Contexto

Esta história é a segunda etapa do pipeline. O Job 1 já consolidou e publicou na staging as entradas necessárias para o cálculo. O Job 2 transforma essas entradas em colunas de negócio prontas para segmentação posterior.

Dependência direta:

- H-01 concluída com staging disponível em `s3://cartao-margem/staging/{ano}/{mes}/`

Esta história habilita:

- H-03, que fará margem líquida final, segmentação e publicação em Redshift/S3

## 3. Entradas Obrigatórias

### 3.1 Tabelas / arquivos de origem

| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| `intercambio` | Parquet | `s3://cartao-margem/staging/{ano}/{mes}/intercambio/` | receita média de intercâmbio por CPF |
| `core` | Parquet | `s3://cartao-margem/staging/{ano}/{mes}/core/` | saldo devedor médio, limite aprovado e score |
| `pdd` | Parquet | `s3://cartao-margem/staging/{ano}/{mes}/pdd/` | faixas de score e percentual de PDD |
| `params.json` | JSON | `s3://cartao-margem/params/params.json` | custo operacional e taxa de juros |

### 3.2 Campos por origem

| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| `intercambio` | `nr_cpf` | string | Sim | chave principal de join |
| `intercambio` | `vl_receita_intercambio` | decimal | Sim | RN01 |
| `core` | `nr_cpf` | string | Sim | chave principal de join |
| `core` | `vl_saldo_devedor_medio` | decimal | Sim | RN02 |
| `core` | `vl_limite_aprovado` | decimal | Sim | RN03 |
| `core` | `nr_score_credito` | int | Sim | join com faixa de PDD |
| `pdd` | `score_min` | int | Sim | limite inferior da faixa |
| `pdd` | `score_max` | int | Sim | limite superior da faixa |
| `pdd` | `pc_pdd` | decimal | Sim | RN03 |
| `params.json` | `vl_custo_operacional` | decimal | Sim | RN04 |
| `params.json` | `taxa_juros_rotativo` | decimal | Sim | RN02 |

## 4. Regras que devem virar código

| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | Receita de intercâmbio já vem consolidada do Job 1 | usar `vl_receita_intercambio` como coluna final sem recalcular | `intercambio.vl_receita_intercambio` |
| RN02 | Receita de juros é saldo devedor médio vezes taxa de juros rotativo | criar `vl_receita_juros`; se `vl_saldo_devedor_medio = 0`, então `vl_receita_juros = 0` | `core.vl_saldo_devedor_medio`, `params.json.taxa_juros_rotativo` |
| RN03 | Custo de PDD é limite aprovado vezes percentual da faixa de score | fazer join por faixa de score e criar `vl_custo_pdd = vl_limite_aprovado * pc_pdd` | `core.vl_limite_aprovado`, `core.nr_score_credito`, `pdd.score_min`, `pdd.score_max`, `pdd.pc_pdd` |
| RN04 | Custo operacional é valor fixo parametrizado | criar `vl_custo_operacional` com valor lido do arquivo de parâmetros | `params.json.vl_custo_operacional` |
| Exclusão | Cliente com score sem faixa válida não deve seguir | excluir registro quando `pc_pdd` for nulo; registrar evento com CPF mascarado | `core.nr_score_credito`, `pdd.pc_pdd`, `core.nr_cpf` |

## 5. Fluxo de Implementação

1. Ler `intercambio`, `core` e `pdd` da staging da partição de execução.
2. Ler `params.json` no início da execução.
3. Fazer join entre `intercambio` e `core` por `nr_cpf`.
4. Fazer join da base consolidada com `pdd` usando regra de faixa:
   `nr_score_credito >= score_min AND nr_score_credito <= score_max`
5. Excluir clientes sem faixa válida de PDD.
6. Criar as colunas:
   `vl_receita_juros`
   `vl_custo_pdd`
   `vl_custo_operacional`
   `vl_receita_total`
   `vl_custo_total`
   `vl_margem_projetada`
7. Persistir o resultado em:
   `s3://cartao-margem/calculado/{ano}/{mes}/`
8. A gravação deve sobrescrever a partição da execução, sem duplicar registros.

## 6. Saída Esperada

### 6.1 Dataset/tabela de saída

| Campo de saída | Origem | Regra |
|---|---|---|
| `nr_cpf` | `intercambio.nr_cpf` / `core.nr_cpf` | chave de saída |
| `vl_receita_intercambio` | `intercambio.vl_receita_intercambio` | RN01 |
| `vl_receita_juros` | derivado | RN02 |
| `vl_custo_pdd` | derivado | RN03 |
| `vl_custo_operacional` | derivado | RN04 |
| `vl_receita_total` | derivado | `vl_receita_intercambio + vl_receita_juros` |
| `vl_custo_total` | derivado | `vl_custo_pdd + vl_custo_operacional` |
| `vl_margem_projetada` | derivado | `vl_receita_total - vl_custo_total` |
| `nr_score_credito` | `core.nr_score_credito` | auditoria e rastreabilidade |
| `vl_limite_aprovado` | `core.vl_limite_aprovado` | auditoria e rastreabilidade |

### 6.2 Critérios de aceite técnicos

- o job deve falhar se qualquer dataset obrigatório não existir
- o job deve falhar se `params.json` não contiver `vl_custo_operacional` ou `taxa_juros_rotativo`
- o output deve conter as colunas individuais, não apenas a margem final
- reexecução para a mesma partição deve sobrescrever sem duplicar
- clientes sem faixa válida de PDD não podem aparecer no output final
- nenhum log pode imprimir CPF sem mascaramento

## 7. Restrições

- não criar novos campos fora dos definidos no Tech Spec e na RFC
- não recalcular RN01 dentro do Job 2; ela já vem pronta do Job 1
- não usar valores hardcoded para juros ou custo operacional; sempre ler de `params.json`
- se houver lacuna de dados, registrar premissa ou falhar explicitamente

## 8. Entregáveis de código

- implementação do Job 2 em Spark
- leitura dos parâmetros via JSON no S3
- join por faixa de score
- persistência em Parquet na área `calculado`
- testes unitários para RN02, RN03 e RN04
- teste de integração com massa de exemplo da staging
- log estruturado para exclusão de clientes sem faixa válida

## 9. Estimativa

| Bloco | Esforço |
|---|---|
| leitura das entradas e parâmetros | 0,5 dia |
| joins e tratamento de faixa de score | 0,5 dia |
| implementação das regras RN01 a RN04 | 1 dia |
| persistência e idempotência por partição | 0,5 dia |
| testes unitários | 0,5 dia |
| teste de integração | 0,5 dia |
| ajuste de logs e tratamento de erro | 0,5 dia |
| total estimado | 4 dias úteis |

### Tamanho sugerido

- Tamanho: `M`
- Faixa: 3 a 5 dias úteis

---

## Como usar este documento como prompt

Você pode copiar a seção do exemplo e entregar diretamente para um agente de código com um comando como:

```text
Implemente a história abaixo exatamente como descrita.
Não invente regras fora da RFC.
Use apenas as tabelas, campos, joins, regras e saídas definidas.
Se faltar alguma informação crítica, registre a premissa no código e no comentário final.
Entregue também testes unitários e uma validação mínima de integração.

[colar aqui o bloco "Prompt de Implementação"]
```

---

## Recomendação Final de Padrão

Para automação, o melhor formato não é um TDD longo e genérico. O melhor formato é:

1. contexto curto
2. inventário de fontes
3. mapeamento de campos
4. regras que viram código
5. saída esperada
6. critérios de aceite
7. estimativa

Esse formato preserva o que há de melhor em Google, Microsoft e Meta:

- clareza de decisão
- separação por história/processo
- schema e lineage explícitos
- contrato técnico pronto para implementação
