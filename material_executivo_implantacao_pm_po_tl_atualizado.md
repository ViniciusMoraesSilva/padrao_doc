# Proposta executiva — Padrão de trabalho entre PM/PO, Tech Lead e Squad

## Mensagem central

O objetivo deste modelo não é criar burocracia. O objetivo é melhorar a passagem de contexto entre negócio e tecnologia para que a squad trabalhe com mais clareza, menos retrabalho e histórias mais maduras para sprint.

O fluxo proposto da squad passa a ser:

`PRD -> RFC -> TDD`

Neste modelo, as histórias **não são uma etapa paralela ao TDD**. As histórias são **derivadas do TDD**, junto com a estratégia de execução da entrega.

---

## 1. Por que mudar agora

Hoje, parte do tempo da squad ainda é consumido tentando descobrir informações que deveriam chegar mais maduras no refinamento, como:

- origem funcional da informação
- sistemas envolvidos
- tabelas, arquivos, entidades e campos relevantes
- regras de negócio que dependem desses dados
- responsáveis funcionais por validar essas informações

Quando isso não chega claro:

- o time técnico assume discovery funcional no lugar do PM/PO
- o Tech Lead vira ponto de concentração
- o refinamento fica mais lento
- as histórias entram com lacunas na sprint
- aumenta o risco de retrabalho, atraso e interpretação errada

No contexto de migração de legado para AWS, esse problema fica ainda mais visível, porque as demandas envolvem regras críticas, integrações, reconciliação e dependências entre áreas.

---

## 2. A decisão proposta

Adotar um padrão único de passagem de contexto e detalhamento da entrega:

### Etapa 1 — PRD

O `PRD` organiza o problema de negócio, o objetivo, o escopo, a origem da informação, as regras de negócio e os critérios de aceite.

### Etapa 2 — RFC

A `RFC` transforma o que foi aprovado no PRD em solução técnica: arquitetura, fluxo, componentes, contrato técnico, qualidade, observabilidade e riscos.

### Etapa 3 — TDD

O `TDD` detalha como a implementação deve acontecer: entradas, saídas, regras técnicas, testes, rollout e quebra da execução.

### Importante

No padrão proposto, o `TDD` já inclui:

- plano de implementação
- sequência sugerida de execução
- critério de pronto
- base para sizing
- derivação das histórias

Ou seja: a história nasce do TDD, e não fora dele.

---

## 3. O que cada artefato resolve

| Artefato | Para que existe | Saída principal |
|---|---|---|
| `PRD` | Organizar o contexto funcional e a necessidade do negócio | Problema, objetivo, escopo, origem da informação, regras e critérios de aceite |
| `RFC` | Definir a solução técnica a partir do PRD | Arquitetura, fluxo técnico, componentes, contratos, qualidade e riscos |
| `TDD` | Detalhar a implementação e preparar a execução | Regras técnicas, estratégia de testes, rollout e histórias derivadas |

### Leitura simples

- o `PRD` responde **o que precisa ser resolvido e por quê**
- a `RFC` responde **qual solução técnica será adotada**
- o `TDD` responde **como vamos implementar, testar e quebrar a entrega**

---

## 4. Como o modelo funciona na prática

### PM/PO é responsável por trazer

- problema de negócio
- objetivo da entrega
- escopo
- critérios de aceite
- regras de negócio
- origem funcional da informação
- sistemas, tabelas, arquivos, entidades e campos conhecidos
- dono funcional para validação

### Tech Lead e time são responsáveis por transformar isso em

- solução técnica
- arquitetura
- integração
- processamento
- qualidade
- estratégia de testes
- observabilidade
- rollout e rollback
- detalhamento da implementação no TDD
- derivação das histórias a partir do TDD

### Princípio de fronteira

Quando houver dúvida sobre responsabilidade, usar a regra:

- se a dúvida é sobre problema, regra, contexto, origem ou dono da informação, a liderança é do PM/PO
- se a dúvida é sobre implementação, arquitetura, integração, teste, rollout ou quebra técnica, a liderança é do Tech Lead e do time

---

## 5. Benefícios esperados

### Mais qualidade de backlog

As histórias passam a nascer com mais contexto, melhor critério de aceite e melhor base para sizing.

### Menos retrabalho

Reduz o número de histórias que voltam por lacuna de regra, origem de dado ou falta de clareza na implementação.

### Refinamento mais rápido

O tempo gasto pelo time técnico com discovery funcional diminui.

### Menos dependência do Tech Lead

O TL deixa de ser o único tradutor entre negócio e tecnologia.

### Mais previsibilidade para sprint

As demandas chegam mais completas antes de entrar em execução.

### Mais autonomia do time

O time participa da construção da solução e da quebra técnica com base em um artefato claro.

### Melhor rastreabilidade

Fica mais fácil conectar:

- decisão de negócio no `PRD`
- solução técnica na `RFC`
- execução e histórias no `TDD`

---

## 6. Papéis e responsabilidades

| Etapa | PM/PO | Tech Lead | Time |
|---|---|---|---|
| `PRD` | Responsável | Consultado | Consultado |
| Handoff para técnico | Responsável | Responsável | Consultado |
| `RFC` | Consultado | Responsável | Responsável |
| `TDD` | Consultado | Responsável | Responsável |
| Derivação das histórias a partir do TDD | Consultado | Responsável | Responsável |
| Criação das histórias na ferramenta | Responsável | Consultado | Informado |
| Sizing | Consultado | Responsável | Responsável |

### Leitura correta dessa tabela

- o PM/PO continua dono do backlog
- o PM/PO continua criando as histórias na ferramenta
- o PM/PO continua responsável pelo contexto funcional
- o Tech Lead e o time continuam responsáveis pela solução técnica
- a quebra das histórias acontece a partir do TDD
- o Tech Lead não deve virar gargalo operacional do processo

---

## 7. Como seguir a estrutura na prática

### Uma demanda começa no PRD

O `PRD` precisa consolidar contexto funcional suficiente para que o time técnico não precise descobrir a regra de negócio do zero.

### A RFC só começa com base madura

A `RFC` deve começar quando já estiver claro:

- qual problema está sendo resolvido
- qual regra de negócio vale
- qual origem funcional da informação será usada
- quem valida dúvidas do contexto funcional

### O TDD detalha a execução

O `TDD` deve transformar a RFC em material implementável, incluindo:

- contrato de entrada
- contrato de saída
- regras de implementação
- estratégia de testes
- rollout
- critério de pronto
- quebra que dá origem às histórias

### Regra operacional simples

- não levar para `RFC` o que ainda é dúvida funcional relevante
- não quebrar histórias fora do contexto do `TDD`
- não levar para sprint o que ainda depende de discovery material

---

## 8. Critérios mínimos por etapa

### PRD está pronto quando

- o problema de negócio está claro
- o objetivo da entrega está claro
- o escopo está definido
- a origem funcional da informação está identificada
- as regras de negócio principais estão descritas
- os critérios de aceite estão explícitos

### RFC está pronta quando

- a solução técnica está definida
- o fluxo técnico está claro
- os componentes e contratos estão identificados
- os riscos e regras de qualidade estão mapeados
- a estratégia de observabilidade e testes está descrita

### TDD está pronto quando

- a implementação está detalhada
- os contratos de entrada e saída estão claros
- as regras técnicas estão explicitadas
- a estratégia de testes está definida
- o rollout está descrito
- as histórias derivadas estão identificadas

---

## 9. O que muda na qualidade das histórias

No modelo atual, a história muitas vezes nasce antes de a implementação estar suficientemente madura.

No modelo proposto, a história nasce depois de existir um `TDD` que já respondeu:

- o que será implementado
- como será implementado
- como será testado
- qual é a ordem da execução
- o que precisa estar pronto para considerar a entrega concluída

Resultado:

- histórias menores
- menos ambiguidade
- melhor sizing
- dependências mais visíveis
- menos retrabalho durante desenvolvimento

---

## 10. Como evitar atrito entre PM/PO e Tech Lead

Esse modelo só funciona bem se for apresentado como uma melhoria de clareza e colaboração, e não como cobrança entre áreas.

### O que este modelo não é

- não é burocracia por burocracia
- não é tentativa de empurrar responsabilidade
- não é exigir que o PM/PO faça papel técnico
- não é colocar o Tech Lead como dono do discovery funcional

### O que este modelo é

- um acordo de interface entre negócio e tecnologia
- uma forma de reduzir ruído e retrabalho
- uma maneira de dar previsibilidade para a sprint
- um mecanismo para não deixar o TL como gargalo

### Mensagem-chave para alinhamento

> O PM/PO não precisa desenhar a solução técnica. Mas precisa trazer o contexto funcional completo da necessidade. O Tech Lead e o time não precisam descobrir a regra de negócio no lugar do PM/PO. Mas precisam transformar esse contexto em uma solução técnica viável, segura, implementável e quebrável em histórias a partir do TDD.

---

## 11. Plano de adoção sugerido

### Semana 1

- apresentar o modelo para PM/PO, TL e squad
- alinhar responsabilidade de cada etapa
- validar o uso dos templates de `PRD`, `RFC` e `TDD`

### Semana 2

- pilotar em 1 demanda real
- usar o fluxo completo `PRD -> RFC -> TDD`
- derivar as histórias a partir do TDD

### Semana 3

- coletar feedback da squad
- ajustar o que ficou excessivo, ambíguo ou pesado

### Semana 4

- oficializar como padrão da squad
- definir quando usar fluxo completo e quando aceitar uma versão simplificada

---

## 12. Indicadores para acompanhar se deu certo

- redução de dúvidas no refinamento técnico
- redução de histórias devolvidas por falta de contexto
- redução de dependência direta do TL para explicar escopo
- aumento de histórias prontas que entram na sprint sem retrabalho
- redução de lacunas funcionais descobertas durante desenvolvimento

---

## 13. Riscos da implantação

| Risco | Mitigação |
|---|---|
| Percepção de aumento de burocracia | Começar com piloto pequeno e manter documentos curtos e objetivos |
| Percepção de conflito entre PM/PO e TL | Explicitar a fronteira entre contexto funcional e solução técnica |
| TL continuar centralizando a passagem de contexto | Fazer o time participar da RFC, do TDD e da derivação das histórias desde o início |

---

## 14. Recomendação final

Recomenda-se implantar este modelo como padrão da squad, começando por um piloto controlado, porque ele melhora a qualidade do backlog, reduz retrabalho e cria uma interface mais saudável entre PM/PO e tecnologia.

O ganho principal não é produzir mais documentos. O ganho principal é fazer a informação chegar mais madura para a solução técnica e permitir que a sprint receba histórias realmente derivadas de um `TDD` claro.

---

## 15. Mensagem final para apresentação

> Esta proposta organiza a passagem de contexto entre PM/PO, Tech Lead e time de forma mais clara. O PM/PO continua dono do backlog e do contexto funcional. O Tech Lead e o time continuam donos da solução técnica. O TDD passa a ser o ponto em que a implementação é detalhada e as histórias são derivadas. Com isso, reduzimos retrabalho, evitamos concentração no TL e melhoramos a qualidade das histórias que entram na sprint.
