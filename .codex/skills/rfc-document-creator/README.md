# RFC Document Creator

Skill para criar RFCs técnicas no padrão da squad a partir de PRD, contexto técnico ou documentos de referência.

## O que faz

Transforma contexto técnico em um documento claro de solução, com:

- problema técnico
- decisão adotada
- escopo técnico
- arquitetura
- contrato de dados
- fluxo principal
- observabilidade
- testes
- plano de implementação
- estimativa total
- riscos
- rollback

## Como usar

**Criar RFC a partir de um PRD**

```text
Leia este PRD e gere uma RFC técnica no padrão da squad.
```

**Criar RFC a partir de contexto técnico**

```text
Transforme este contexto em uma RFC técnica enxuta e implementável.
```

**Evoluir uma RFC existente**

```text
Atualize esta RFC preservando o que continua válido e refletindo as novas decisões técnicas.
```

## O que a skill garante

| Área | Regra aplicada |
|---|---|
| Decisão | Explicita a escolha técnica e seu papel |
| Arquitetura | Organiza componentes, fluxo e limites da solução |
| Dados | Define entrada, saída, campos e regras técnicas quando aplicável |
| Operação | Inclui falha, rejeição, reprocessamento e observabilidade |
| Execução | Traz testes, plano de implementação em história única, estimativa total, riscos e rollback |

## Observações

- A skill é orientada a solução técnica, não a discovery de negócio.
- Se faltar base para uma decisão, a RFC deve registrar a lacuna explicitamente.
- O documento final deve ser técnico, mas curto e utilizável pelo time de implementação.
- O plano de implementação deve ser consolidado em uma única história com estimativa total explícita.
