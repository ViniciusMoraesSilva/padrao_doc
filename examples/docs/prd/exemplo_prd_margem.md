# PRD — Entrega de Variáveis de Margem Gerencial para Contratos Imobiliários

**PM/PO:** Mariana Costa · **Data:** 21/04/2026 · **Área:** Margem Gerencial

---

## Em uma frase

Disponibilizar, para o sistema de margem gerencial do banco, uma visão mensal de
contratos imobiliários com as chaves do contrato e 10 variáveis de valor padronizadas.

---

## Contexto do problema

Hoje a apuração de margem gerencial para contratos imobiliários não parte de uma saída
padronizada. As áreas acabam dependendo de extrações manuais, tratamentos paralelos e
interpretações diferentes sobre os mesmos contratos.

Na prática, isso gera três problemas:

1. números diferentes entre áreas para a mesma carteira
2. tempo excessivo para fechamento gerencial
3. baixa rastreabilidade sobre quais contratos e quais valores foram considerados

Precisamos de uma entrega simples e confiável para que o sistema de margem gerencial
receba sempre a mesma estrutura, no mesmo padrão, todos os meses.

---

## Objetivo do produto

Entregar uma tabela mensal, pronta para consumo, contendo:

- as chaves de identificação do contrato
- 10 variáveis de valor por contrato
- valores consistentes e comparáveis entre meses

O foco desta iniciativa não é criar uma tela nova nem uma consulta online. O foco é
garantir que o dado certo chegue ao sistema de margem gerencial de forma estável.

---

## Para quem é essa entrega

**Sistema de Margem Gerencial**
> "Preciso receber uma estrutura única, previsível e sem tratamento manual para calcular
> e consolidar a margem da carteira."

O que precisa: uma tabela final padronizada, com uma linha por contrato.

**Controladoria**
> "Quero confiar que o número usado no fechamento veio da mesma regra para toda a base."

O que precisa: consistência e rastreabilidade mensal.

**Produtos Imobiliários**
> "Preciso acompanhar a contribuição financeira da carteira sem depender de ajustes
> manuais a cada fechamento."

O que precisa: visão organizada por contrato, com variáveis de negócio claras.

---

## Escopo

### O que está dentro do escopo

| Item | Prioridade |
|---|---|
| Receber como entrada uma base com somente contratos imobiliários | Essencial |
| Entregar as chaves do contrato para identificação do registro | Essencial |
| Entregar 10 variáveis de valor por contrato | Essencial |
| Disponibilizar uma saída mensal pronta para consumo pelo sistema de margem gerencial | Essencial |
| Garantir padrão único de estrutura entre execuções mensais | Essencial |

### O que não está no escopo desta versão

| Item | Motivo |
|---|---|
| Outros produtos além de imobiliário | Esta entrega é exclusiva para carteira imobiliária |
| Consulta online contrato a contrato | A necessidade atual é batch mensal |
| Ajuste manual de registros na própria solução | Correções permanecem na origem |
| Regras específicas por subproduto imobiliário | Fica para evolução futura, se necessário |

---

## Jornada esperada

### Situação atual

As áreas recebem dados de contratos imobiliários, mas ainda precisam interpretar,
reorganizar e complementar informações antes de consumir no processo gerencial.

### Situação desejada

Todo mês, o sistema de margem gerencial passa a receber uma saída pronta, com:

- uma linha por contrato
- chaves de identificação do contrato
- 10 variáveis de valor no mesmo padrão

Isso reduz retrabalho, evita divergência de entendimento e acelera o uso do dado.

---

## Origem da informação

A entrada desta iniciativa será uma tabela que contém somente contratos imobiliários.

Premissas de negócio:

- a origem já vem filtrada para produto imobiliário
- cada registro representa um contrato elegível para processamento
- a qualidade mínima da origem continua sendo responsabilidade do sistema produtor

### Fonte de dados de entrada

| O que preciso | Tabela / sistema | Campos utilizados | Quem confirmou | Status | Observação |
|---|---|---|---|---|---|
| Identificação do contrato | `tb_contrato_imobiliario_origem` | `cd_contrato`, `cd_sistema_origem`, `nr_proposta` | Juliana Rocha | Validado em 18/04/2026 | Uso obrigatório na chave da saída |
| Identificação do cliente | `tb_contrato_imobiliario_origem` | `nr_cpf_cnpj_cliente` | Juliana Rocha | Validado em 18/04/2026 | Validar ainda política de mascaramento |
| Datas do contrato | `tb_contrato_imobiliario_origem` | `dt_contratacao`, `dt_vencimento_final` | Ricardo Menezes | Validado em 19/04/2026 | Base para prazo remanescente |
| Saldo do contrato | `tb_contrato_imobiliario_origem` | `vl_saldo_devedor_atual` | Ricardo Menezes | Validado em 19/04/2026 | Base para saldo médio da V1 |
| Parcela do contrato | `tb_contrato_imobiliario_origem` | `vl_parcela_atual` | Ricardo Menezes | Validado em 19/04/2026 | Base para receita de tarifas da V1 |
| Taxa do contrato | `tb_contrato_imobiliario_origem` | `vl_taxa_juros_aa` | Patrícia Souza | Validado em 20/04/2026 | Base para receita de juros |
| Garantia do imóvel | `tb_contrato_imobiliario_origem` | `vl_garantia_imovel` | Patrícia Souza | Validado em 20/04/2026 | Base para cálculo de LTV |
| Funding | `tb_contrato_imobiliario_origem` | `vl_custo_funding_aa` | Eduardo Lima | Validado em 20/04/2026 | Base para custo de funding |
| Perda esperada | `tb_contrato_imobiliario_origem` | `pc_inadimplencia_modelo` | Fernanda Alves | Validado em 20/04/2026 | Base para cálculo de perda esperada |
| Custo operacional | `tb_contrato_imobiliario_origem` | `pc_custo_operacional` | Fernanda Alves | Validado em 20/04/2026 | Base para custo operacional |

### Regra de preenchimento desta seção

Quem escreve o PRD deve informar:

- qual tabela será usada
- quais campos da origem serão consumidos
- quem validou a existência e o significado desses campos
- se a informação está validada, pendente ou com restrição conhecida

Se algum campo necessário ainda não estiver validado, isso deve aparecer explicitamente no
documento antes do handoff para o time técnico.

---

## Chaves que devem ser entregues

Cada contrato deve ser identificado na saída pelas seguintes chaves:

| Chave | Finalidade |
|---|---|
| `cd_contrato` | Identificação única do contrato |
| `cd_sistema_origem` | Identificação do sistema de origem |
| `nr_proposta` | Rastreabilidade comercial do contrato |
| `nr_cpf_cnpj_cliente` | Identificação do titular do contrato |
| `dt_referencia` | Competência mensal da informação |

---

## Variáveis de valor que o sistema deve receber

O sistema de margem gerencial deve receber exatamente 10 variáveis de valor por contrato:

| Variável | Leitura de negócio |
|---|---|
| `vl_saldo_medio` | Valor representativo do saldo do contrato no período |
| `vl_margem_liquida` | Resultado líquido final associado ao contrato |
| `vl_receita_juros` | Receita financeira do contrato |
| `vl_custo_funding` | Custo de funding associado ao contrato |
| `vl_custo_operacional` | Custo operacional atribuído ao contrato |
| `vl_perda_esperada` | Valor esperado de perda do contrato |
| `vl_receita_tarifas` | Receita de tarifas associada ao contrato |
| `vl_resultado_bruto` | Resultado antes de custos finais e perdas |
| `vl_ltv_atual` | Relação entre saldo do contrato e valor da garantia |
| `vl_prazo_remanescente_meses` | Prazo restante do contrato em meses |

---

## Regras de negócio

### Regra 1

A saída deve conter uma linha para cada contrato imobiliário válido recebido na entrada.

### Regra 2

Todo contrato da saída deve trazer, obrigatoriamente, as chaves de identificação e as 10
variáveis de valor definidas neste documento.

### Regra 3

`vl_saldo_medio` e `vl_margem_liquida` são variáveis obrigatórias da entrega e devem
estar presentes em todos os contratos válidos processados.

### Regra 4

Quando um contrato não tiver informação mínima para cálculo ou identificação, ele não deve
ser publicado como registro válido na saída final.

### Regra 5

Quando a garantia do contrato não permitir cálculo confiável da relação com o saldo,
o valor de `vl_ltv_atual` poderá ser entregue em branco, desde que isso não impeça a
publicação das demais variáveis do contrato.

### Regra 6

Quando o contrato já estiver vencido na data de referência, o prazo remanescente deve ser
considerado igual a zero.

### Regra 7

A estrutura da saída deve permanecer estável entre os meses, para evitar quebra no sistema
consumidor.

### Regra 8 — Cálculo de `vl_saldo_medio`

`vl_saldo_medio = vl_saldo_devedor_atual`

Para esta versão, o saldo médio será representado pelo saldo devedor atual do contrato,
porque a origem disponibilizada para a iniciativa não possui histórico mensal de saldo.

### Regra 9 — Cálculo de `vl_receita_juros`

`vl_receita_juros = vl_saldo_medio * (vl_taxa_juros_aa / 12)`

### Regra 10 — Cálculo de `vl_custo_funding`

`vl_custo_funding = vl_saldo_medio * (vl_custo_funding_aa / 12)`

### Regra 11 — Cálculo de `vl_custo_operacional`

`vl_custo_operacional = vl_saldo_medio * pc_custo_operacional`

### Regra 12 — Cálculo de `vl_perda_esperada`

`vl_perda_esperada = vl_saldo_medio * pc_inadimplencia_modelo`

### Regra 13 — Cálculo de `vl_receita_tarifas`

`vl_receita_tarifas = vl_parcela_atual * 0,002`

Nesta versão, a receita de tarifas será representada por um fator fixo de 0,2% sobre o
valor da parcela atual.

### Regra 14 — Cálculo de `vl_resultado_bruto`

`vl_resultado_bruto = vl_receita_juros + vl_receita_tarifas - vl_custo_funding`

### Regra 15 — Cálculo de `vl_ltv_atual`

`vl_ltv_atual = vl_saldo_devedor_atual / vl_garantia_imovel`

Quando `vl_garantia_imovel` for menor ou igual a zero, o campo poderá ser entregue em
branco para não comprometer a publicação do contrato.

### Regra 16 — Cálculo de `vl_prazo_remanescente_meses`

`vl_prazo_remanescente_meses = meses entre dt_referencia e dt_vencimento_final`

Quando o contrato já estiver vencido na data de referência, o valor deverá ser zero.

### Regra 17 — Cálculo de `vl_margem_liquida`

`vl_margem_liquida = vl_resultado_bruto - vl_perda_esperada - vl_custo_operacional`

---

## Critérios de aceite

| Situação | Resultado esperado |
|---|---|
| Contrato válido recebido na entrada | Deve existir uma linha correspondente na saída |
| Contrato publicado na saída | Deve conter todas as chaves obrigatórias |
| Contrato publicado na saída | Deve conter as 10 variáveis previstas neste PRD |
| Contrato com problema de identificação | Não deve ser considerado válido para publicação |
| Contrato vencido | Deve ter prazo remanescente igual a zero |
| Contrato válido com saldo devedor atual preenchido | `vl_saldo_medio` deve ser calculado conforme a regra definida neste PRD |
| Contrato válido com taxa de juros preenchida | `vl_receita_juros` deve ser calculada conforme a regra definida neste PRD |
| Contrato válido com custo de funding preenchido | `vl_custo_funding` deve ser calculado conforme a regra definida neste PRD |
| Contrato válido com percentual de custo operacional preenchido | `vl_custo_operacional` deve ser calculado conforme a regra definida neste PRD |
| Contrato válido com percentual de inadimplência preenchido | `vl_perda_esperada` deve ser calculada conforme a regra definida neste PRD |
| Contrato válido com parcela preenchida | `vl_receita_tarifas` deve ser calculada conforme a regra definida neste PRD |
| Contrato válido com garantia positiva | `vl_ltv_atual` deve ser calculado conforme a regra definida neste PRD |
| Contrato com garantia menor ou igual a zero | `vl_ltv_atual` pode ser entregue em branco |
| Contrato válido publicado | `vl_margem_liquida` deve refletir a composição prevista neste PRD |
| Execução de um novo mês | Deve manter exatamente a mesma estrutura de colunas |

---

## Resultado esperado para o negócio

Ao final desta entrega, o banco passa a contar com uma visão mensal padronizada da
carteira imobiliária para uso no sistema de margem gerencial.

O benefício esperado é:

- reduzir reconciliações manuais entre áreas
- aumentar confiança no dado de margem
- acelerar fechamento e análise gerencial
- criar base consistente para evoluções futuras de rentabilidade imobiliária

---

## Métricas de sucesso

| Métrica | Meta inicial |
|---|---|
| Disponibilização da saída no ciclo mensal acordado | 100% |
| Registros entregues fora do padrão esperado pelo consumidor | 0 |
| Divergência estrutural entre meses | 0 |
| Necessidade de tratamento manual após entrega | Redução relevante vs. processo atual |

---

## Dependências e alinhamentos

| Tema | Necessidade |
|---|---|
| Origem de contratos imobiliários | Confirmar estabilidade e completude mínima da base |
| Sistema consumidor | Validar estrutura final esperada |
| Controladoria | Validar aderência das variáveis ao uso gerencial |
| Segurança da Informação | Confirmar política para documento do cliente na saída |

---

## Perguntas em aberto

| Questão | Responsável | Prazo | Impacto |
|---|---|---|---|
| O documento do cliente deve ser entregue aberto ou mascarado? | Segurança da Informação | 25/04/2026 | Alto |
| A leitura de saldo médio desta versão, usando saldo devedor atual, atende à expectativa da Controladoria? | Controladoria | 25/04/2026 | Médio |
| Haverá necessidade de segmentação futura por subproduto imobiliário? | Produtos Imobiliários | 30/04/2026 | Baixo |

---

## Handoff

- [x] Problema de negócio está claro
- [x] Objetivo da entrega está claro
- [x] Entrada está definida em linguagem de negócio
- [x] Tabela e campos de origem foram informados
- [x] Status de validação das informações foi registrado
- [x] Chaves do contrato estão definidas
- [x] As 10 variáveis estão nomeadas
- [x] Regras de cálculo dos campos derivados estão explícitas
- [ ] Pendências de negócio foram resolvidas
