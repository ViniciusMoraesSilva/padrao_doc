# Apresentação — Implantação do Fluxo `Product Requirements Document (PRD) -> Request for Comments (RFC) -> Technical Design Document (TDD)`

## Slide 1 — Título e mensagem central

**Como melhorar a passagem de contexto entre PM/PA, TL e Squad**

**Mensagem central**

O objetivo deste modelo não é criar burocracia.

O objetivo é melhorar a passagem de contexto entre negócio e tecnologia para que a squad trabalhe com mais clareza, mais previsibilidade e histórias mais maduras para sprint.

**Fluxo proposto**

`Product Requirements Document (PRD) -> Request for Comments (RFC) -> Technical Design Document (TDD)`

`PRD` = `Product Requirements Document`

`RFC` = `Request for Comments`

`TDD` = `Technical Design Document`

**Fala sugerida**

Hoje a proposta é alinhar um modelo simples de trabalho entre PM/PA, Tech Lead e squad. A ideia não é aumentar documento, e sim fazer a informação chegar mais madura para a execução. Por isso, já vale começar deixando claro o que significam `Product Requirements Document (PRD)`, `Request for Comments (RFC)` e `Technical Design Document (TDD)` dentro desse fluxo.

---

## Slide 2 — O que o modelo nos ajuda a evitar

**Com esse modelo, vamos evitar**

- perda de tempo consolidando contexto em etapas mais tardias do fluxo
- decisões importantes sendo amadurecidas tarde demais
- concentração excessiva de contexto em uma única pessoa
- histórias chegando à sprint com níveis diferentes de maturidade
- perda de previsibilidade na passagem entre planejamento e execução

**Por que isso pesa mais agora**

No contexto de migração de legado para AWS, as demandas têm mais dependências, integrações e regras críticas. Por isso, quanto mais cedo o contexto estiver alinhado, maior a chance de termos execução fluida, decisões mais rápidas e sprint previsível.

**Fala sugerida**

A proposta não parte de um problema de pessoas, e sim de uma oportunidade de fortalecer o processo. A ideia é dar mais suporte para que PM/PA, TL e squad trabalhem com a mesma base de contexto desde o início.

---

## Slide 3 — A decisão proposta

**Adotar um padrão único de passagem de contexto**

### `Product Requirements Document (PRD)`

Organiza o que precisa ser resolvido e por quê.

### `Request for Comments (RFC)`

Transforma o que foi aprovado em solução técnica.

### `Technical Design Document (TDD)`

Detalha como implementar, testar, fazer rollout e estruturar a execução da entrega.

**Evolução principal**

O `Technical Design Document (TDD)` passa a representar a própria história de implementação, com contexto técnico suficiente para priorização, alinhamento e sizing.

**Fala sugerida**

O fluxo passa a ser contínuo. Primeiro entendemos o problema de negócio, depois definimos a solução técnica, e só então detalhamos a implementação em um `Technical Design Document (TDD)` que já representa a história a ser executada.

---

## Slide 4 — O que significam `Product Requirements Document (PRD)`, `Request for Comments (RFC)` e `Technical Design Document (TDD)`

| Sigla | Nome | Leitura prática |
|---|---|---|
| `PRD` | `Product Requirements Document` | Documento que organiza o problema de negócio, objetivo, escopo, regras e critérios de aceite |
| `RFC` | `Request for Comments` | Documento que descreve a solução técnica proposta para validação e alinhamento |
| `TDD` | `Technical Design Document` | Documento que detalha a implementação, testes, rollout e a estrutura de execução da história |

**Importante**

Aqui, `TDD` significa `Technical Design Document`, e não `Test-Driven Development`.

**Fala sugerida**

Essas três siglas representam etapas complementares. Elas ajudam a organizar melhor o contexto funcional, a solução técnica e o detalhamento da execução.

---

## Slide 5 — O que cada artefato resolve

| Artefato | Para que existe | Saída principal |
|---|---|---|
| `Product Requirements Document (PRD)` | Organizar o contexto funcional e a necessidade do negócio | Problema, objetivo, escopo, origem da informação, regras e critérios de aceite |
| `Request for Comments (RFC)` | Definir a solução técnica a partir do `Product Requirements Document (PRD)` | Arquitetura, fluxo técnico, componentes, contratos, qualidade e riscos |
| `Technical Design Document (TDD)` | Detalhar a implementação e preparar a execução | Regras técnicas, estratégia de testes, rollout e estrutura de execução da história |

**Leitura simples**

- o `Product Requirements Document (PRD)` responde o que precisa ser resolvido e por quê
- a `Request for Comments (RFC)` responde qual solução técnica será adotada
- o `Technical Design Document (TDD)` responde como vamos implementar, testar e executar a entrega

**Fala sugerida**

Essa separação ajuda a dar mais clareza ao fluxo. Cada artefato existe para resolver uma pergunta diferente, sem misturar contexto funcional com decisão de implementação.

---

## Slide 6 — Papéis e fronteiras

| Etapa | PM/PA | Tech Lead | Time |
|---|---|---|---|
| `Product Requirements Document (PRD)` | Responsável | Consultado | Consultado |
| Handoff para técnico | Responsável | Responsável | Consultado |
| `Request for Comments (RFC)` | Consultado | Responsável | Responsável |
| `Technical Design Document (TDD)` | Consultado | Responsável | Responsável |
| Construção da história técnica no `Technical Design Document (TDD)` | Consultado | Responsável | Responsável |
| Organização do backlog na ferramenta | Responsável | Consultado | Consultado |
| Sizing | Consultado | Responsável | Responsável |

**Divisão saudável de foco**

- contexto de negócio, objetivo, regra e validação funcional: protagonismo de `PM/PA`
- arquitetura, integração, implementação, testes e rollout: protagonismo de `TL` e time

**Mensagem-chave**

O `PM/PA` continua dono do backlog, da priorização e do direcionamento funcional.

O `Tech Lead` e o time continuam responsáveis por transformar esse contexto em solução técnica viável.

O modelo ajuda a distribuir melhor o contexto e evita concentração desnecessária em uma única pessoa.

**Fala sugerida**

Esse modelo reforça a parceria entre produto e tecnologia. Ele deixa mais claro como cada papel contribui para a mesma entrega, com mais alinhamento e fluidez.

---

## Slide 7 — Como funciona na prática

**Fluxo operacional**

1. A demanda começa no `Product Requirements Document (PRD)`
2. A `Request for Comments (RFC)` começa com uma base funcional já alinhada
3. O `Technical Design Document (TDD)` detalha a execução e consolida a história técnica

**Como esse fluxo apoia a execução**

- o `Product Requirements Document (PRD)` ajuda a consolidar o contexto da necessidade
- a `Request for Comments (RFC)` organiza a direção da solução técnica
- o `Technical Design Document (TDD)` reúne o detalhamento necessário para execução com mais clareza e previsibilidade

**O papel do `Technical Design Document (TDD)`**

Ele concentra os principais elementos da implementação, como escopo técnico, forma de execução, estratégia de testes, critérios de pronto e visão de rollout.

**Fala sugerida**

Na prática, a principal mudança é que o `Technical Design Document (TDD)` passa a ser o ponto de convergência entre contexto, solução e execução. Isso fortalece a qualidade da história técnica e dá mais segurança para planejamento, priorização e sizing.

---

## Slide 8 — Benefícios esperados

**Ganhos para produto e tecnologia**

- mais qualidade de backlog
- mais clareza na execução
- refinamento mais rápido
- menos dependência de tradução informal entre áreas
- mais previsibilidade para sprint
- mais autonomia do time
- melhor rastreabilidade entre decisão de negócio, solução técnica e execução

**Resultado esperado**

Histórias menores, mais claras, com dependências mais visíveis e mais confiança na passagem do planejamento para a execução.

**Fala sugerida**

O ganho principal não é produzir mais documentos. O ganho principal é dar ao PM/PA, ao TL e à squad uma base comum mais forte para priorizar, refinar e executar com segurança.

---

## Slide 9 — Plano de adoção e fechamento

**Piloto sugerido**

- **Semana 1:** apresentar o modelo, alinhar responsabilidades e validar templates
- **Semana 2:** pilotar em uma demanda real usando `Product Requirements Document (PRD) -> Request for Comments (RFC) -> Technical Design Document (TDD)`
- **Semana 3:** coletar feedback da squad e ajustar o que puder ficar mais leve e simples
- **Semana 4:** oficializar o padrão e definir quando usar versão completa ou simplificada

**Cuidados na adoção**

- manter os documentos objetivos e proporcionais ao tamanho da demanda
- reforçar desde o início que o objetivo é colaboração, previsibilidade e clareza compartilhada
- envolver o time desde `Request for Comments (RFC)` e `Technical Design Document (TDD)` para distribuir contexto

**Mensagem final**

Esta proposta fortalece a parceria entre `PM/PA`, `Tech Lead` e time com uma passagem de contexto mais clara.

Com isso, aumentamos previsibilidade e melhoramos a qualidade das histórias técnicas que entram na sprint.

**Chamada para ação**

Vamos pilotar em uma demanda real e ajustar juntos.

**Fala sugerida**

Se o modelo funcionar, produto e tecnologia ganham mais clareza, o backlog melhora e a sprint passa a receber histórias mais maduras. A proposta é começar leve, testar e evoluir em conjunto.
