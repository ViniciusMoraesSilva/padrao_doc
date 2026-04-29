Matriz de responsabilidade — PM, PA e Tech Lead no Discovery

| Etapa / Entregável                              | PM                                                                                                                                                                      | PA                                                                                                                   | Tech Lead/Time                                                                                                                                            |
| ----------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Contexto do produto/processo**             | **Dono.** Explica o que é o EZIS/RN0 no contexto de negócio, qual processo suporta, quem usa e por que existe.                                                          | Apoia com dados de uso, volumetria, recorrência, impactos e evidências.                                              | Consultado para explicar limitações ou funcionamento técnico quando necessário.                                                                      |
| **2. Problema ou oportunidade**                 | **Dono.** Define qual problema de negócio está sendo resolvido e qual oportunidade está sendo perseguida.                                                               | Apoia com análises que comprovem a dor, impacto, frequência e relevância.                                            | Consultado para avaliar se o problema tem impacto técnico, dependências ou riscos relevantes.                                                        |
| **3. Visão funcional AS IS**                    | **Dono.** Deve entender e descrever como o processo funciona hoje funcionalmente, de ponta a ponta.                                                                     | Apoia com evidências, dados históricos, reconciliações e comportamento observado.                                    | Apoia apenas quando for necessário explicar como o legado implementa determinado comportamento.                                                      |
| **4. Fluxo de informação de negócio**           | **Dono.** Explica quais informações entram, por que entram, como são usadas, quais regras funcionais são aplicadas e qual saída é esperada.                             | Apoia mapeando dados, campos, indicadores, amostras, exceções e impactos.                                            | Consultado para informar onde a informação nasce tecnicamente, onde trafega, onde é transformada e quais sistemas são impactados.                    |
| **5. Regras funcionais**                        | **Dono.** Define e valida as regras de negócio com as áreas responsáveis. Exemplo: “valor de prêmio entra, aplica regras X/Y/Z e gera o resultado esperado para o RN0”. | Apoia validando se os dados confirmam a regra, se existem exceções, desvios ou comportamento histórico diferente.    | Não define regra funcional. Apenas informa o comportamento identificado no legado e alerta divergências, riscos ou lacunas.                          |
| **6. Visão funcional TO BE**                    | **Dono.** Define como o processo deveria funcionar no futuro, quais regras permanecem, mudam ou deixam de existir.                                                      | Apoia com simulações, impacto esperado e comparação entre cenário atual e futuro.                                    | Consultado para avaliar viabilidade, esforço, restrições, riscos técnicos e alternativas de implementação.                                           |
| **7. Métricas de sucesso / valor**              | **Dono.** Define qual valor o produto/processo deve gerar: redução de erro, eficiência, prazo, receita, risco, qualidade etc.                                           | **Dono da medição.** Define baseline, indicadores, dashboards, critérios de acompanhamento e leitura dos resultados. | Apoia garantindo que a solução gere logs, rastreabilidade, observabilidade e dados necessários para medição.                                         |
| **8. Origem e qualidade dos dados**             | Responsável por dizer qual informação é necessária e para qual finalidade de negócio.                                                                                   | **Dono analítico.** Avalia qualidade, completude, consistência, amostras, reconciliação e aderência dos dados.       | Consultado para explicar fonte técnica, linhagem, transformações, integrações e limitações do legado.                                                |
| **9. Validação com stakeholders de negócio**    | **Dono.** Conduz validação com áreas usuárias, produto, operações, contabilidade, risco, seguros ou demais áreas impactadas.                                            | Apoia com dados e evidências para facilitar a tomada de decisão.                                                     | Participa quando houver dúvida técnica ou necessidade de explicar comportamento implementado.                                                        |
| **10. Lacunas funcionais**                      | **Dono.** Identifica perguntas abertas de negócio e busca resposta com os responsáveis funcionais.                                                                      | Apoia analisando dados para esclarecer dúvidas e identificar exceções.                                               | Aponta lacunas técnicas, mas não assume lacunas funcionais como responsabilidade dele.                                                               |
| **11. Leitura técnica do legado**               | Consultado para entender impacto funcional.                                                                                                                             | Consultado para cruzar comportamento técnico com dados.                                                              | **Dono.** Explica como o legado implementa o processo: programas, jobs, arquivos, tabelas, dependências, cálculos encontrados, integrações e riscos. |
| **12. Comportamento implementado no mainframe** | Valida se o comportamento encontrado representa regra oficial, exceção ou legado obsoleto.                                                                              | Apoia com evidência de dados e impacto dos comportamentos encontrados.                                               | **Dono da evidência técnica.** Mostra o que o código faz hoje, sem transformar isso automaticamente em regra funcional oficial.                      |
| **13. Riscos e dependências técnicas**          | Considera os riscos na priorização e no plano de produto.                                                                                                               | Apoia estimando impacto nos indicadores ou operação.                                                                 | **Dono.** Identifica riscos, dependências, integrações, restrições, impactos sistêmicos, performance, segurança, observabilidade e sustentação.      |
| **14. Priorização**                             | **Dono.** Decide prioridade com base em valor, urgência, estratégia, risco e alinhamento com stakeholders.                                                              | Apoia com dados, impacto e cenários.                                                                                 | Consultado para informar esforço, risco técnico, dependências e complexidade.                                                                        |
| **15. Critérios de aceite funcionais**          | **Dono.** Define o comportamento esperado do ponto de vista de negócio.                                                                                                 | Apoia com exemplos, massas, cenários, exceções e validações.                                                         | Apoia transformando critérios funcionais em critérios técnicos/testáveis, mas não é dono da regra funcional.                                         |
| **16. Critérios técnicos de aceite**            | Informado.                                                                                                                                                              | Informado.                                                                                                           | **Dono.** Define critérios de qualidade técnica, performance, rastreabilidade, logs, monitoração, rollback, segurança e sustentação.                 |
| **17. Desenho da solução**                      | Define necessidade, objetivo e escopo funcional.                                                                                                                        | Apoia com dados e cenários.                                                                                          | **Dono técnico.** Define arquitetura, desenho técnico, estratégia de implementação, integrações, padrões e alternativas.                             |
| **18. Decisão final sobre regra de negócio**    | **Dono com Negócio.** Decide e valida a regra oficial.                                                                                                                  | Apoia com evidências e análises.                                                                                     | Consultado. Pode alertar: “no legado está diferente”, mas não decide qual regra funcional deve prevalecer.                                           |
| **19. Pós-go-live**                             | Acompanha se o valor de negócio foi alcançado.                                                                                                                          | Mede resultado, compara baseline e monitora indicadores.                                                             | Acompanha estabilidade técnica, performance, incidentes, logs, rastreabilidade e débitos técnicos.                                                   |

---

Resumo por papel

PM — Dono da visão funcional e de produto

O PM deve ser responsável por responder:

* O que é o processo/produto?
* Quem usa?
* Qual problema estamos resolvendo?
* Qual valor queremos gerar?
* Quais informações são necessárias?
* Quais regras funcionais precisam existir?
* Qual é a jornada ponta a ponta?
* Qual sistema consome a saída e por quê?
* O que deve permanecer, mudar ou deixar de existir no TO BE?
* Quais decisões funcionais precisam ser tomadas?

No seu exemplo, o PM deveria conseguir explicar algo como:

“Hoje o processo consome o valor de prêmio de um seguro. Esse valor passa pelas regras funcionais X, Y e Z para gerar uma informação final que será enviada para o RN0. O RN0 consome esse campo porque ele é necessário para determinada finalidade de negócio.”

⸻

PA — Dono da evidência, dados e medição

O PA deve ser responsável por apoiar com:

* volumetria;
* amostras;
* comportamento histórico;
* qualidade dos dados;
* análise de exceções;
* indicadores;
* baseline;
* impacto esperado;
* validação quantitativa das regras;
* dashboards;
* evidências para tomada de decisão.

O PA ajuda a responder:

“Os dados mostram que essa regra acontece assim?”
“Quantos casos são impactados?”
“Qual o impacto se a regra mudar?”
“Existem exceções?”
“A informação está consistente?”
“Como vamos medir se deu certo?”

⸻

Tech Lead/Time — Dono da viabilidade e evidência técnica

O Tech Lead/Time deve ser responsável por:

* entender como o legado implementa o comportamento atual;
* explicar dependências técnicas;
* mapear integrações;
* identificar origem técnica dos dados;
* apontar riscos;
* avaliar viabilidade;
* propor arquitetura;
* definir estratégia técnica;
* orientar implementação;
* garantir qualidade, segurança, performance e sustentação.

Mas o Tech Lead/Time não deve ser o dono de definir a regra funcional.

Você pode dizer:

“Eu posso mostrar que no legado o cálculo hoje faz A, B e C. Mas quem precisa validar se A, B e C são regras oficiais de negócio, exceções históricas ou comportamentos que devem mudar no TO BE é Produto/Negócio.”

⸻

Matriz RACI simplificada

Legenda:
R = Responsible / Executa
A = Accountable / Dono final
C = Consulted / Consultado
I = Informed / Informado

---

| Atividade                                              |  PM |  PA | Tech Lead/Time |
| ------------------------------------------------------ | --: | --: | --------: |
| Definir problema de negócio                            | A/R |   C |         C |
| Construir visão funcional AS IS                        | A/R |   C |         C |
| Construir visão funcional TO BE                        | A/R |   C |         C |
| Mapear regras funcionais                               | A/R |   C |         C |
| Validar regra com negócio                              | A/R |   C |       I/C |
| Mapear campos necessários e finalidade                 | A/R |   C |         C |
| Analisar dados, volumetria e exceções                  |   C | A/R |         C |
| Definir métricas de sucesso                            |   A |   R |         C |
| Explicar comportamento técnico do legado               |   C |   C |       A/R |
| Identificar divergência entre regra funcional e legado |   A | R/C |         R |
| Definir arquitetura da solução                         |   C |   C |       A/R |
| Avaliar viabilidade técnica                            |   C |   C |       A/R |
| Definir estratégia de implementação                    |   C |   I |       A/R |
| Definir prioridade                                     | A/R |   C |         C |
| Definir critérios de aceite funcionais                 | A/R |   C |         C |
| Definir critérios técnicos                             |   I |   I |       A/R |
| Aprovar regra funcional final                          | A/R |   C |         C |
| Garantir observabilidade e sustentação                 |   I |   C |       A/R |


---

# Frase para você usar na reunião

Eu estou totalmente à disposição para ajudar como Tech Lead e o time tambem, principalmente para explicar o comportamento técnico do legado, origem das informações, dependências, riscos, viabilidade e impactos da solução. Mas eu preciso deixar claro que a definição das regras funcionais, da jornada de negócio e da visão ponta a ponta do processo precisa ser conduzida por Produto/Negócio, com apoio do PA. O que eu trago do mainframe é evidência técnica de como o sistema se comporta hoje; isso não deve ser assumido automaticamente como regra funcional oficial nem como definição do TO BE sem validação de negócio.

## Uma versão mais curta e firme:

Eu ajudo como consultado técnico para explicar legado, origem dos dados, riscos e viabilidade. Mas a definição da regra funcional, da necessidade de negócio e da visão ponta a ponta do processo precisa ser responsabilidade de Produto/Negócio. O legado mostra o comportamento implementado hoje, não necessariamente a regra oficial que deve seguir no TO BE.

## E a frase mais executiva:

Meu papel é garantir viabilidade técnica e trazer evidências do legado; não definir sozinho regras funcionais ou assumir a visão de negócio do processo.
