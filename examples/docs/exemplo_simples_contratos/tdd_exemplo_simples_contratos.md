# TDD - História técnica de transformação simples de contratos

**TL / Tech Lead:** Exemplo didático · **Responsável técnico:** Exemplo didático · **Data:** 21/04/2026
**PRD de origem:** `prd_exemplo_simples_contratos.md` · **RFC de origem:** `rfc_exemplo_simples_contratos.md` · **Status:** Rascunho

> Este documento detalha a história técnica de implementação do exemplo simples de contratos.

## 1. Objetivo da História

Implementar um processo simples que leia `tb_contratos_origem`, consuma `chave_contrato`, `valor_1`, `valor_2` e `valor_3`, calcule `valor_calculado_1` e `valor_calculado_2`, e publique a saída final em `tb_contratos_saida`.

## 2. Contexto

Esta história representa a implementação técnica do exemplo usado na apresentação da squad.

Ela existe para mostrar, de forma simples, como uma necessidade da área de acompanhamento comercial, descrita no `Product Requirements Document (PRD)` e organizada na `Request for Comments (RFC)`, vira uma instrução implementável.

O negócio já definiu a tabela de origem, os campos necessários e a granularidade de 1 linha por contrato.

Não há dependências técnicas anteriores além da existência da tabela de entrada.

## 3. Entradas Obrigatórias

### 3.1 Tabelas / arquivos de origem

| Origem | Tipo | Local | Finalidade |
|---|---|---|---|
| `tb_contratos_origem` | tabela | banco/schema do exemplo | leitura dos campos necessários para entregar a visão de contratos definida pelo negócio |

### 3.2 Campos por origem

| Origem | Campo | Tipo esperado | Obrigatório | Uso na regra |
|---|---|---|---|---|
| `tb_contratos_origem` | `chave_contrato` | string | Sim | chave da saída |
| `tb_contratos_origem` | `valor_1` | decimal(18,2) | Sim | cálculo de `valor_calculado_1` |
| `tb_contratos_origem` | `valor_2` | decimal(18,2) | Sim | cálculo de `valor_calculado_1` e `valor_calculado_2` |
| `tb_contratos_origem` | `valor_3` | decimal(18,2) | Sim | cálculo de `valor_calculado_2` |

## 4. Regras que devem virar código

| Regra | Descrição funcional | Implementação esperada | Entradas |
|---|---|---|---|
| RN01 | A saída deve conter uma linha para cada contrato | manter a granularidade de 1 linha por contrato na publicação final | `tb_contratos_origem.chave_contrato` |
| RN02 | A saída deve trazer a `chave_contrato` e os campos `valor_1`, `valor_2` e `valor_3` | copiar os campos originais da origem para a saída | `tb_contratos_origem.chave_contrato`, `valor_1`, `valor_2`, `valor_3` |
| RN03 | O campo `valor_calculado_1` deve ser gerado por multiplicação | criar `valor_calculado_1 = valor_1 * valor_2` | `tb_contratos_origem.valor_1`, `valor_2` |
| RN04 | O campo `valor_calculado_2` deve ser gerado por divisão | criar `valor_calculado_2 = valor_3 / valor_2` | `tb_contratos_origem.valor_3`, `valor_2` |
| RN05 | Quando `valor_2 = 0`, o contrato deve permanecer na saída e `valor_calculado_2` não deve trazer valor | preencher `valor_calculado_2` com nulo sem rejeitar o registro | `tb_contratos_origem.valor_2` |
| RT01 | Registros sem `chave_contrato` não devem seguir para a saída técnica | rejeitar o registro antes da publicação | `chave_contrato` |
| RT02 | Registros sem `valor_1`, `valor_2` ou `valor_3` não devem seguir para a saída técnica | rejeitar o registro antes da publicação | `valor_1`, `valor_2`, `valor_3` |

## 5. Fluxo de Implementação

1. Ler `tb_contratos_origem`.
2. Validar a existência de `chave_contrato`, `valor_1`, `valor_2` e `valor_3`.
3. Rejeitar registros com `chave_contrato` ausente conforme RT01.
4. Rejeitar registros com `valor_1`, `valor_2` ou `valor_3` ausentes conforme RT02.
5. Garantir que a saída continue com 1 linha por contrato conforme RN01.
6. Copiar `chave_contrato`, `valor_1`, `valor_2` e `valor_3` para a saída conforme RN02.
7. Criar `valor_calculado_1` com a regra RN03.
8. Criar `valor_calculado_2` com a regra RN04.
9. Aplicar RN05 quando `valor_2 = 0`, mantendo o registro e preenchendo `valor_calculado_2` com nulo.
10. Persistir a saída em `tb_contratos_saida`.

## 6. Saída Esperada

### 6.1 Dataset/tabela de saída

| Campo | Origem | Regra |
|---|---|---|
| `chave_contrato` | `tb_contratos_origem.chave_contrato` | cópia direta |
| `valor_1` | `tb_contratos_origem.valor_1` | cópia direta |
| `valor_2` | `tb_contratos_origem.valor_2` | cópia direta |
| `valor_3` | `tb_contratos_origem.valor_3` | cópia direta |
| `valor_calculado_1` | derivado | RN03 |
| `valor_calculado_2` | derivado | RN04 e RN05 |

### 6.2 Critérios de aceite técnicos

- não publicar registro sem `chave_contrato`
- não publicar registro sem `valor_1`, `valor_2` ou `valor_3`
- manter a granularidade de 1 linha por contrato
- copiar para a saída `chave_contrato`, `valor_1`, `valor_2` e `valor_3`
- calcular `valor_calculado_1` com multiplicação simples
- calcular `valor_calculado_2` com divisão simples
- manter o registro na saída quando `valor_2 = 0`
- publicar `valor_calculado_2` sem valor quando `valor_2 = 0`
- publicar a tabela final com exatamente os 6 campos previstos

## 7. Restrições

- não inventar campos além dos definidos no `Product Requirements Document (PRD)` e na `Request for Comments (RFC)`
- não criar regras adicionais fora das fórmulas descritas
- manter a implementação simples e compatível com o objetivo didático do exemplo

## 8. Entregáveis de código

- processo de leitura de `tb_contratos_origem`
- transformação com criação de `valor_calculado_1` e `valor_calculado_2`
- publicação da tabela `tb_contratos_saida`
- testes unitários das regras
- teste de contrato da saída

## 9. Estimativa

| Bloco | Esforço |
|---|---|
| leitura da entrada | 0,25 dia |
| validação dos campos obrigatórios | 0,25 dia |
| implementação das regras | 0,5 dia |
| persistência da saída | 0,25 dia |
| testes | 0,5 dia |
| total | 1,75 dia |
