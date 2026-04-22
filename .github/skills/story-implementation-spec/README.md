# Story Implementation Spec

Skill para gerar TDDs operacionais por história a partir de RFC, PRD, Tech Spec ou histórias já detalhadas.

## O que faz

Transforma documentação funcional em um contrato enxuto para implementação assistida por IA, sempre deixando claro:

- objetivo da história
- entradas obrigatórias
- campos por origem
- regras implementáveis
- fluxo operacional
- saída esperada
- restrições
- entregáveis
- estimativa

## Como usar

**RFC para TDD por história**

```text
Leia esta RFC e gere um TDD operacional por história.
```

**História para TDD implementável**

```text
Transforme esta história em um TDD operacional enxuto para implementação.
```

**Continuação de TDD**

```text
Atualize este TDD com base na história nova, preservando o que continua válido e adicionando só o delta.
```

## O que a skill garante

| Área | Regra aplicada |
|---|---|
| Fontes | Sempre mapeia tabela, arquivo, dataset ou API de origem |
| Campos | Sempre liga campo à origem e ao uso na regra |
| Regras | Traduz requisito em regra implementável, sem inventar comportamento |
| Fluxo | Explicita leitura, join, filtro, cálculo e persistência |
| Saída | Define artefato final, campos e critérios técnicos de aceite |
| Lacunas | Registra premissas explicitamente em vez de preencher no chute |
| Estimativa | Quebra esforço por bloco e informa total |

## Como a consistência é mantida

- A skill já traz um template mínimo embutido no `SKILL.md`.
- Ela inclui um exemplo curto de saída esperada para ancorar o formato.
- Ela também define explicitamente o que não fazer, para evitar respostas genéricas ou conceituais demais.

## Estrutura do documento gerado

1. Objetivo da História
2. Contexto
3. Entradas Obrigatórias
4. Regras que devem virar código
5. Fluxo de Implementação
6. Saída Esperada
7. Restrições
8. Entregáveis de código
9. Estimativa

## Observações

- A skill é autossuficiente e não depende de `prompt_tdd_automatizacao_historia.md`.
- Se faltar informação crítica, a resposta deve registrar premissas explicitamente.
- O documento final deve ser curto, preciso e orientado à execução.
