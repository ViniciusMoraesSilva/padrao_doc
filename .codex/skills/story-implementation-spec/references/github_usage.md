# Uso no GitHub

Use este texto-base quando quiser aplicar a mesma skill em contexto de GitHub, issue, comentário ou conversa com IA:

```md
Leia a RFC ou história informada e gere ou atualize um TDD operacional no mesmo padrão de `prompt_tdd_automatizacao_historia.md`.

O documento precisa:

1. ter contexto curto e objetivo
2. listar tabelas e arquivos de origem
3. listar campos por origem
4. transformar regras da RFC em regras implementáveis
5. deixar explícito o fluxo: leitura, join, filtro, cálculo e persistência
6. detalhar a saída esperada
7. incluir critérios de aceite técnicos
8. incluir restrições
9. incluir estimativa por bloco e total

Não invente regra fora da RFC, PRD, Tech Spec ou história.
Se faltar informação, registre a premissa explicitamente.
```

## Fluxo recomendado

### 1. RFC -> história

Use quando quiser gerar um documento por história.

### 2. História -> continuação do TDD

Use quando já existe um TDD e você quer que a IA continue a evolução do documento sem perder o contexto anterior.

## Regra de ouro

Se houver conflito entre um pedido curto no GitHub e o documento-base, prevalece o padrão de:

- `prompt_tdd_automatizacao_historia.md`
