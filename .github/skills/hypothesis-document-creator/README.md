# Hypothesis Document Creator

Skill para criar Documentos de Hipótese completos seguindo o padrão da Squad.

## O que faz

Conduz o PO/PM por todas as 10 seções do Documento de Hipótese através de perguntas interativas,
ou extrai as informações a partir de um documento de referência fornecido (contexto, dados de analytics,
e-mail de stakeholder, anotações de discovery etc.).

Garante que o documento resultante:
- Tenha hipótese no formato correto (Se/Então/Porque)
- Tenha métrica com baseline, meta, guardrail e janela de medição
- Identifique a origem inicial dos dados (tabelas, campos, responsáveis)
- Passe no Gate de aprovação antes de avançar para PRD

## Como usar

**Com documento de referência:**
> "Cria um documento de hipótese com base nesse contexto: [colar texto / anexar arquivo]"

**Do zero com perguntas:**
> "Cria um documento de hipótese para [nome do projeto/ideia]"

**Misto:**
> "Tenho essas anotações de discovery [texto]. Ajuda a montar o documento de hipótese fazendo perguntas para o que estiver faltando."

## Exemplos de prompts

```
Cria um documento de hipótese para o projeto de pontos do cartão
```

```
Preciso documentar uma hipótese sobre abandono no onboarding. Tenho esses dados: [colar]
```

```
Quero avançar para PRD mas ainda não tenho a hipótese documentada — me ajuda?
```

## O que a skill garante

| Seção | Regra aplicada |
|---|---|
| Hipótese | Sempre no formato Se/Então/Porque — nunca aceita hipótese sem "porque" |
| Métrica | Exige nome + baseline + meta + guardrail + janela — sinaliza o que estiver faltando |
| Origem dos dados | Sempre identifica pelo menos sistema/área + responsável + lacunas |
| Gate | Marca [x] o que está completo e [ ] o que ainda precisa ser resolvido |

## Estrutura do documento gerado

1. Problema observado
2. Hipótese de solução (Se/Então/Porque)
3. Evidências e contexto
4. Métrica de sucesso (baseline → meta + guardrail + janela)
5. Prazo para validação
6. Riscos e premissas
7. Impacto estimado
8. Esforço percebido (Pequeno / Médio / Grande)
9. Fora do escopo
10. Origem inicial da informação
Gate de aprovação PM/PO + TL
