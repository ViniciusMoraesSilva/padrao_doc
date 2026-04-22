---
name: story-implementation-spec
description: Gera TDDs operacionais por história a partir de RFCs e continua a evolução desses TDDs a partir de histórias. Use quando precisar ler uma RFC, quebrar por história, mapear tabelas, campos, regras e saídas, e produzir especificações prontas para implementação por IA no Codex ou reutilização no GitHub.
---

# Story Implementation Spec

Use esta skill para transformar uma RFC em documentos operacionais por história, no formato de TDD orientado a automação, e para continuar a evolução desses documentos quando a história mudar.

## Quando usar

- quando o usuário pedir para ler uma RFC e gerar um TDD por história
- quando o usuário pedir para transformar uma história em um documento mais implementável por IA
- quando o usuário pedir para atualizar um TDD existente com base em uma história
- quando for importante explicitar tabelas, campos, joins, regras, saídas e estimativa

## Fonte de verdade

Leia primeiro:

- `prompt_tdd_automatizacao_historia.md`

Se existirem, também leia:

- RFCs, PRDs e Tech Specs mencionados pelo usuário
- histórias derivadas da RFC
- TDDs anteriores da mesma feature

## Objetivo do documento gerado

O resultado final não deve ser apenas um TDD descritivo. Ele deve servir como contrato operacional para implementação assistida por IA.

O documento precisa permitir frases objetivas como:

- “o programa deve ler a tabela X”
- “a regra RN01 usa os campos Y e Z”
- “o join deve ocorrer pela chave K”
- “a saída deve ser persistida em Z”
- “a reexecução deve sobrescrever a partição”

## Estrutura obrigatória

Todo documento gerado por esta skill deve conter:

1. Objetivo da História
2. Contexto
3. Entradas Obrigatórias
4. Regras que devem virar código
5. Fluxo de Implementação
6. Saída Esperada
7. Restrições
8. Entregáveis de código
9. Estimativa

## Regras obrigatórias

- escrever na mesma língua do pedido do usuário
- não inventar regras fora da RFC, PRD, Tech Spec ou história
- sempre mapear tabela ou arquivo de origem
- sempre mapear campos por origem
- sempre dizer qual campo participa de qual regra
- sempre explicitar joins, filtros e exclusões quando existirem
- sempre explicitar a tabela, dataset ou arquivo de saída
- sempre incluir estimativa por bloco e total
- se houver lacuna crítica, registrar a premissa explicitamente

## Workflow

### 1. Ler e extrair

Extrair do material de origem:

- nome e objetivo da história
- dependências
- tabelas e arquivos de entrada
- campos por origem
- regras de negócio
- critérios de aceite
- saídas esperadas
- riscos, premissas e lacunas

### 2. Converter em contrato operacional

Transformar o conteúdo em uma instrução implementável:

- origem -> campos -> regra -> saída

Cada regra deve deixar claro:

- de onde vem cada campo
- qual join ou filtro precisa existir
- qual coluna ou artefato deve ser criado
- onde o resultado deve ser persistido

### 3. Estimar

Sempre estimar por blocos:

- leitura das entradas
- joins e filtros
- implementação das regras
- persistência
- testes
- observabilidade ou tratamento de erro, se aplicável

Também classificar um tamanho:

- `PPP`, `PP`, `P`, `M`, `G`, `GG`, `XG`, `XXG`

## Modos de uso

### Modo A: RFC -> TDD por história

Saída esperada:

- um documento por história
- todos no formato operacional acima

### Modo B: História -> continuação do TDD

Saída esperada:

- atualização incremental do TDD existente
- novos campos, regras, saídas e estimativa ajustados

## Continuação de TDD existente

Quando o pedido for continuar ou evoluir um TDD:

- leia primeiro o TDD atual
- compare com a história nova ou alterada
- preserve o que continua válido
- adicione apenas o delta necessário
- atualize estimativa, regras, entradas e saídas

## Critério de pronto

Não considerar pronto se faltar:

- origem de dados
- campos obrigatórios
- pelo menos uma regra traduzida para implementação
- saída esperada
- critérios técnicos de aceite
- estimativa

## Uso no GitHub

Para reutilizar o mesmo padrão no GitHub, leia:

- `references/github_usage.md`

Essa referência contém um texto-base curto para colar em issue, comentário ou conversa com IA no GitHub, sem mudar o contrato principal desta skill.
