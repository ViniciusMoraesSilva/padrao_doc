# Padrão PM/PO + Tech Lead + Time para Hipótese, PRD, Tech Spec e Histórias

## Objetivo
Definir um fluxo simples, claro e escalável entre PM/PO, Tech Lead e time para transformar uma hipótese em histórias prontas para sprint, sem fazer o TL virar gargalo e sem deixar o time técnico responsável por descobrir regra de negócio no lugar do PM/PO.

---

## 1. Resumo executivo do modelo

### Princípio central
- **PM/PO** é responsável por entender e documentar o problema, o objetivo, as regras de negócio e o caminho funcional da informação.
- **Tech Lead + time** são responsáveis por desenhar a solução técnica, validar viabilidade no handoff, quebrar tecnicamente a entrega e apoiar o refinamento.
- **PO/PM continua sendo o responsável por criar as histórias na ferramenta**.
- **A quebra das histórias acontece em conjunto entre PO/PM + TL + time**.

### Regra prática
- O **PO/PM cria a história** no board/ferramenta.
- O **TL e o time ajudam a definir a estrutura técnica da quebra**, dependências, granularidade, riscos e sizing.
- O **time não deve descobrir sozinho regra de negócio, tabela, campo, entidade ou origem funcional que o PM/PO deveria ter trazido**.

---

## 2. Fluxo proposto da squad

### Fase 1 — Hipótese
**Responsável principal:** PM/PO  
**Objetivo:** explicar problema, valor, hipótese, métrica, risco e origem inicial da informação.

**Saída:** Documento de Hipótese.

### Gate 1 — Discovery / Validação
**Responsável principal:** PM/PO  
**Participação do TL:** consultiva, quando necessário.

**Objetivo:** validar se a hipótese é relevante, testável e se já existe base funcional suficiente para evoluir para PRD.

**Possíveis decisões:**
- avançar para PRD
- pedir mais insumos
- repriorizar
- rejeitar

### Fase 2 — PRD funcional-operacional
**Responsável principal:** PM/PO  
**Apoio do TL:** consultivo para dúvidas de viabilidade e preparação do handoff.

**Objetivo:** detalhar escopo, regras de negócio, critérios de aceite, dependências e principalmente o caminho funcional da informação.

**Saída:** PRD pronto para handoff técnico.

### Gate 2 — Handoff para Tech Spec
**Responsáveis:** PM/PO + TL

**Objetivo:** garantir que o PRD tenha informação suficiente para o time técnico começar a desenhar solução sem precisar fazer discovery funcional no lugar do PM/PO.

### Fase 3 — Tech Spec / RFC
**Responsável principal:** TL + time  
**Apoio:** PM/PO para esclarecer regra e contexto.

**Objetivo:** definir arquitetura, abordagem técnica, integração, estratégia de testes, rollout, rollback, observabilidade e quebra técnica.

**Saída:** Tech Spec revisado.

### Gate 3 — Refinamento técnico e quebra
**Responsáveis:** TL + time + PM/PO

**Objetivo:** transformar o escopo em histórias pequenas, coerentes e prontas para criação no backlog.

### Fase 4 — Histórias e tasks
**Responsável por criar na ferramenta:** PM/PO  
**Responsável por definir a quebra junto:** TL + time + PM/PO

**Objetivo:** registrar as histórias no board, classificar prioridade, detalhar critério de aceite e fazer sizing.

---

## 3. Papéis e responsabilidades

| Etapa | PM/PO | Tech Lead | Time |
|---|---|---|---|
| Documento de hipótese | R | C | I |
| Discovery | R | C | I |
| PRD | R | C | I |
| Handoff funcional para técnico | R | R | C |
| Tech Spec / RFC | C | R | R |
| Refinamento técnico | C | R | R |
| Quebra das histórias | R | R | R |
| Criação das histórias na ferramenta | R | C | I |
| Sizing | C | R | R |
| Entrada em sprint | R | R | R |

**Legenda**
- **R** = responsável direto
- **C** = consultado
- **I** = informado

### Interpretação importante
A **quebra é conjunta**, mas a **criação da história no Jira/Azure/board é do PO/PM**.  
O TL e o time não são os donos operacionais da criação do card; eles ajudam a estruturar corretamente.

### Princípio de responsabilidade
- **Discovery é de Negócio.**
- **O Tech Lead não é responsável pela descoberta do problema.**
- **O TL entra como consultivo quando necessário e como corresponsável apenas no handoff para solução técnica.**

---

## 4. Regras de governança do processo

1. Toda iniciativa começa em **Hipótese**.
2. Nenhuma demanda relevante vai para Tech Spec sem um **PRD funcional-operacional minimamente completo**.
3. O PM/PO não precisa desenhar arquitetura, mas precisa trazer:
   - origem funcional do dado
   - sistema dono
   - tabela / arquivo / entidade conhecida
   - campos relevantes já identificados
   - regra de negócio ligada ao dado
   - responsável funcional pela validação
4. O TL e o time não devem assumir discovery funcional como atividade padrão.
5. O PO/PM cria as histórias na ferramenta após a quebra conjunta.
6. História grande demais não entra em sprint.
7. Toda demanda crítica de migração deve considerar teste, observabilidade, reconciliação e rollback.

---

## 5. Critério de passagem entre fases

### Hipótese → PRD
Só passa se tiver:
- problema claro
- hipótese testável
- métrica e janela de validação
- impacto estimado
- risco principal identificado
- origem inicial da informação minimamente indicada

### PRD → Tech Spec
Só passa se tiver:
- escopo claro
- critério de aceite de negócio
- regras de negócio descritas
- dependências mapeadas
- origem funcional dos dados identificada
- tabelas / arquivos / entidades conhecidas
- campos relevantes já levantados
- dúvidas em aberto com responsável e prazo

### Tech Spec → Histórias
Só passa se tiver:
- solução técnica compreendida pelo time
- riscos com mitigação
- rollout e rollback definidos
- dependências técnicas claras
- observabilidade definida
- quebra inicial validada em conjunto

### Histórias → Sprint
Só entra se cumprir **Definition of Ready**.

---

## 6. Definition of Ready sugerida

Uma história só entra em sprint quando:
- objetivo estiver claro
- regra de negócio estiver clara
- critério de aceite estiver claro
- dependências estiverem identificadas
- não houver dúvida crítica bloqueando implementação
- o tamanho couber na sprint
- a equipe entender o que precisa ser feito sem depender de discovery adicional relevante

---

## 7. Método simples de sizing

### P — Pequena
- até 2 dias úteis
- mudança localizada
- baixa dependência
- baixo risco

### M — Média
- 3 a 5 dias úteis
- alguma dependência
- risco moderado
- exige validação cruzada

### G — Grande
- acima de 5 dias úteis
- vários sistemas / componentes
- alta incerteza
- precisa ser quebrada antes da sprint

**Regra:** história G não entra na sprint.

---

# 8. Template 1 — Documento de Hipótese

```md
# Documento de Hipótese

**Autor:** [nome do PM/PO]  
**Data:** [preencher]  
**Squad:** [preencher]  
**Status:** Rascunho / Em review / Aprovado / Reprioritizado

---

## 1. Problema observado
> Descreva o problema do ponto de vista do usuário, processo ou negócio.

[preencher]

## 2. Hipótese de solução
> Se fizermos [X], acreditamos que [Y] vai acontecer, porque [Z].

[preencher]

## 3. Evidências e contexto
> Dados, pesquisas, incidentes, feedbacks, benchmarks ou insumos que sustentam a hipótese.

- [ ] [evidência 1]
- [ ] [evidência 2]

## 4. Métrica de sucesso
> Baseline atual → Meta esperada.

- Métrica: [nome]
- Baseline: [valor atual]
- Meta: [valor esperado]
- Janela de medição: [ex: 30 dias após lançamento]

## 5. Prazo para validação
[preencher]

## 6. Riscos e premissas
| Premissa / Risco | Probabilidade | Mitigação |
|---|---|---|
| [exemplo] | Alta / Média / Baixa | [ação] |

## 7. Impacto estimado
- Nível: Alto / Médio / Baixo
- Justificativa: [preencher]
- Usuários/processos afetados: [estimativa]

## 8. Esforço percebido (PM/PO)
- Tamanho: Pequeno / Médio / Grande
- Observação: [preencher]

## 9. Fora do escopo
- [item 1]
- [item 2]

## 10. Origem inicial da informação
> Onde o PM/PO acredita que estão as principais informações necessárias para essa iniciativa?

- Sistema(s) provável(is): [preencher]
- Área dona da informação: [preencher]
- Tabela / arquivo / relatório / entidade conhecida: [preencher]
- Pessoa ou time que pode validar: [preencher]
- Lacunas ainda existentes: [preencher]

---

## Gate — PM/PO + apoio consultivo do TL
- [ ] Problema claro e embasado?
- [ ] Hipótese testável com métrica e prazo?
- [ ] Impacto e risco entendidos?
- [ ] Origem inicial da informação minimamente identificada?
- [ ] Já existe base suficiente para evoluir para PRD?
- [ ] Decisão: Avançar para PRD / Pedir mais insumos / Repriorizar
```

---

# 9. Template 2 — PRD funcional-operacional

```md
# PRD — Product Requirements Document

**Hipótese de origem:** [link]  
**Squad:** [nome] · **PM/PO:** [nome] · **TL:** [nome]  
**Data:** [preencher] · **Status:** Rascunho / Em review / Aprovado

---

## 1. Objetivo e contexto
[3–5 frases: qual o objetivo, qual problema resolve, por que agora]

## 2. Personas, usuários ou processos afetados
[Quem será impactado, em que contexto e o que muda]

## 3. Escopo

| Funcionalidade / comportamento | Prioridade | Justificativa |
|---|---|---|
| [item] | Must have / Should have / Won't have | [motivo] |

## 4. Fluxo funcional
[Passo a passo do fluxo principal + link Figma/Miro/processo se houver]

1. [passo 1]
2. [passo 2]
   - 2a. [caminho feliz]
   - 2b. [alternativo]

## 5. Critérios de aceite de negócio

| Dado que... / Quando... / Então... | Tipo |
|---|---|
| Dado que [contexto], quando [ação], então [resultado esperado]. | Happy path |
| Dado que [edge case], quando [ação], então [comportamento esperado]. | Edge case |

## 6. Métricas de sucesso
- **Primária:** [métrica] — baseline: [X] → meta: [Y]
- **Secundária:** [métrica complementar]
- **Guardrail:** [o que não pode piorar]
- **Janela de medição:** [preencher]

## 7. Origem funcional dos dados
> Esta seção é obrigatória quando houver dependência de informação de outros sistemas, times, legado ou bases de dados.

| Informação necessária | Sistema origem | Tabela / arquivo / entidade | Campo(s) relevantes | Chave de referência | Dono da informação | Observações |
|---|---|---|---|---|---|---|
| [informação] | [sistema] | [origem] | [campo 1, campo 2] | [id/chave] | [área/time] | [contexto] |

## 8. Regras de negócio detalhadas com base em dados

| Regra | Descrição funcional | Dados necessários | Fonte funcional | Responsável pela validação |
|---|---|---|---|---|
| RN01 | [descrição] | [campos] | [origem] | [nome/área] |
| RN02 | [descrição] | [campos] | [origem] | [nome/área] |

## 9. Mapeamento AS IS funcional
- Processo atual: [descrever]
- Sistemas envolvidos:
  - [sistema 1]
  - [sistema 2]
- Entradas:
  - [entrada 1]
- Saídas:
  - [saída 1]
- Regras conhecidas no legado:
  - [regra 1]
- Pontos não esclarecidos:
  - [dúvida 1]

## 10. Dicionário funcional mínimo

| Campo | Significado | Regra associada | Obrigatório? | Exemplo |
|---|---|---|---|---|
| [campo] | [significado] | [regra] | Sim / Não | [exemplo] |

## 11. Dependências externas
- Outros times: [preencher]
- APIs / serviços / sistemas: [preencher]
- Prazo externo: [se houver]

## 12. Requisitos não-funcionais (visão de negócio)
- [ex: janela de processamento]
- [ex: rastreabilidade obrigatória]
- [ex: auditoria]

## 13. Perguntas em aberto
- [ ] [questão 1] — responsável: [nome] — prazo: [data]
- [ ] [questão 2]

---

## Gate de handoff para o Tech Lead
- [ ] Escopo Must/Should/Won't acordado com stakeholders?
- [ ] Critérios de aceite cobrem happy paths e edge cases principais?
- [ ] Fluxo funcional e/ou material de apoio disponível?
- [ ] Dependências externas mapeadas?
- [ ] Origem funcional dos dados identificada?
- [ ] Tabelas / arquivos / entidades de referência informados?
- [ ] Campos relevantes para a regra documentados?
- [ ] Donos das informações e regras identificados?
- [ ] Perguntas em aberto resolvidas ou com responsável?
- [ ] TL consegue começar a solução sem precisar descobrir a regra no lugar do PM/PO?
```

---

# 10. Template 3 — Tech Spec / RFC

```md
# Tech Spec / RFC

**PRD de origem:** [link]  
**Tech Lead:** [nome] · **Revisores:** [nomes]  
**Data:** [preencher] · **Status:** Rascunho / Em review / Aprovado · **Versão:** v1

---

## 1. Resumo técnico (TL;DR)
[4–6 frases: o que será construído, abordagem técnica, impacto em sistemas existentes]

## 2. Contexto técnico atual
[Como o sistema funciona hoje nas áreas que serão tocadas. Diagrama se ajudar.]

Componentes envolvidos:
- [serviço / banco / fila / job]
- [serviço / banco / fila / job]

## 3. Insumos recebidos do PRD
- Sistemas origem identificados: [sim/não]
- Tabelas/arquivos informados: [listar]
- Campos mapeados funcionalmente: [listar]
- Regras já validadas pelo negócio: [listar]
- Lacunas remanescentes: [listar]

## 4. Solução proposta
[Arquitetura da solução com detalhe suficiente para o time implementar.]

Novos componentes:
- [componente — responsabilidade]

Mudanças em existentes:
- [componente — o que muda]

Fluxo de dados:
1. [passo 1]
2. [passo 2]

## 5. Alternativas consideradas

| Alternativa | Status | Por que descartada / escolhida |
|---|---|---|
| [opção A] | Escolhida | [justificativa] |
| [opção B] | Descartada | [justificativa] |
| [opção C] | Descartada | [justificativa] |

## 6. Modelo de dados e contratos

### Schema
```sql
-- [nova tabela ou campo]
```

### API
```text
POST /v1/[recurso]
Body: { campo1, campo2 }
Response 201: { id, created_at }
Response 422: { error: "codigo_erro" }
```

### Eventos / mensagens
- [evento publicado na fila X com payload Y]

## 7. Requisitos não-funcionais
- Performance: [exemplo]
- Disponibilidade: [exemplo]
- Escalabilidade: [exemplo]
- Segurança: [exemplo]

## 8. Estratégia de rollout
- Feature flag: Sim / Não — [% inicial e critério para abrir]
- Migração de dados: [script necessário? backfill? janela?]
- Rollback: [como reverter]
- Ordem de deploy: [backend antes de frontend? dados antes do código?]

## 9. Riscos técnicos

| Risco | Severidade | Mitigação |
|---|---|---|
| [risco 1] | Alta / Média / Baixa | [mitigação] |
| [risco 2] | Alta / Média / Baixa | [mitigação] |

## 10. Plano de observabilidade
- Métricas: [o que medir]
- Logs: [eventos a logar]
- Alertas: [condição e threshold]

## 11. Lacunas assumidas pelo técnico
> Itens que não vieram completos no PRD e precisaram ser complementados pelo time técnico.

| Item | Impacto | Ação tomada | Risco |
|---|---|---|---|
| [item] | [alto/médio/baixo] | [ação] | [risco] |

## 12. Proposta de quebra em histórias
> A quebra é construída em conjunto entre TL + time + PM/PO. A criação final das histórias na ferramenta fica com o PM/PO.

| História proposta | Tipo | Dependência |
|---|---|---|
| [história 1] | Backend / Frontend / Dados / Infra / Obs. | [dep. ou —] |
| [história 2] | [tipo] | [dep.] |
| [história 3] | [tipo] | [dep.] |

## 13. Estimativa de esforço

| Área | Complexidade | Observação |
|---|---|---|
| Backend | S / M / L (X dias) | |
| Frontend | S / M / L (X dias) | |
| Dados / Infra | S / M / L (X dias) | |
| QA / testes | S / M / L (X dias) | |

## 14. Perguntas em aberto
- [ ] [questão] — responsável: [nome] — prazo: [data]

---

## Gate de refinamento — TL + time + PM/PO
- [ ] Time entende a solução sem depender exclusivamente do TL?
- [ ] Alternativas descartadas documentadas com justificativa?
- [ ] Riscos com mitigação definida?
- [ ] Ordem de deploy e dependências claras?
- [ ] Estratégia de rollback existe?
- [ ] Proposta de histórias é quebrável em itens de sprint?
- [ ] Plano de observabilidade definido antes de codar?
- [ ] PM/PO tem insumo suficiente para criar as histórias finais no board?
```

---

# 11. Template 4 — História para o board

```md
# História

**Título:** [verbo + resultado]
**Origem:** [link do PRD] / [link do Tech Spec]
**Responsável pela criação no board:** [PM/PO]

## Descrição
Como [tipo de usuário / sistema / operação]
Quero [capacidade]
Para [benefício]

## Contexto
[Resumo curto do porquê essa história existe]

## Critérios de aceite
- [CA1]
- [CA2]
- [CA3]

## Dependências
- [dependência 1]
- [dependência 2]

## Observações técnicas
- [observação 1]
- [observação 2]

## Sizing
P / M / G

## Tasks sugeridas
- [task 1]
- [task 2]
```

---

# 12. Ritual sugerido da squad

### 1. Intake / triagem
30 min
- PM/PO apresenta hipóteses
- TL participa de forma consultiva quando necessário
- decide-se o que precisa evoluir para PRD

### 2. Discovery review
45 min
- revisão da hipótese liderada por negócio
- TL participa apenas como apoio consultivo quando necessário
- decisão de avançar, repriorizar ou devolver

### 3. Refinamento funcional-operacional
45 a 60 min
- PM/PO apresenta PRD
- TL valida se há informação suficiente para iniciar solução

### 4. Review técnico
60 min
- TL + time apresentam solução proposta
- PM/PO tira dúvidas de regra

### 5. Sessão de quebra e sizing
45 min
- TL + time + PM/PO quebram as histórias
- PM/PO sai com insumo para criar os cards no board

### 6. Planejamento de sprint
- entram apenas histórias que cumpriram Definition of Ready

---

# 13. Resumo final para implantação

## O que muda na prática
- O **PM/PO continua criando as histórias**.
- A **quebra das histórias passa a ser conjunta**.
- O **Discovery continua sendo de Negócio**.
- O **PRD deixa de ser só funcional** e passa a ser **funcional-operacional**.
- O **Tech Spec só começa quando o PM/PO trouxer contexto suficiente**.
- O **TL deixa de ser gargalo**, porque o time participa da solução e da quebra.

## Regra de bolso
- **PM/PO traz o que, por quê, regra e origem funcional da informação**.
- **TL + time trazem como implementar**.
- **PM/PO registra as histórias finais na ferramenta**.
- **TL participa do Discovery apenas como consultivo quando necessário**.

## Melhor frase para alinhar com a squad
> O objetivo desse processo não é burocratizar. É garantir que o problema, a regra e a origem da informação cheguem claros para o time técnico, e que a solução saia com qualidade sem concentrar tudo no TL. O PM/PO continua dono do backlog e da criação das histórias. O Discovery continua sendo de negócio. O TL e o time entram para transformar isso em solução implementável, com boa quebra, bom sizing e menor risco.
