# Exemplo simples — contratos

Este pacote foi criado para apoiar a apresentação do fluxo:

`Product Requirements Document (PRD) -> Request for Comments (RFC) -> Technical Design Document (TDD)`

## Cenário do exemplo

Temos uma tabela de contratos de origem.

Para cada contrato, precisamos:

- buscar a `chave_contrato`
- buscar `valor_1`, `valor_2` e `valor_3`
- calcular `valor_calculado_1`
- calcular `valor_calculado_2`
- publicar uma tabela final com os campos originais e os campos calculados

## Regras do exemplo

- `valor_calculado_1 = valor_1 * valor_2`
- `valor_calculado_2 = valor_3 / valor_2`
- quando `valor_2 = 0`, `valor_calculado_2` não deve ser calculado

## Como usar na apresentação

### 1. Comece pelo `Product Requirements Document (PRD)`

Mostre que ele explica a necessidade de negócio, a origem da informação, os campos e as regras.

### 2. Depois apresente a `Request for Comments (RFC)`

Mostre que ela transforma a necessidade em solução técnica: leitura, validação, cálculo e publicação.

### 3. Feche com o `Technical Design Document (TDD)`

Mostre que ele detalha a história técnica de implementação com entradas, regras que viram código, testes e saída esperada.

## Arquivos

- `prd_exemplo_simples_contratos.md`
- `rfc_exemplo_simples_contratos.md`
- `tdd_exemplo_simples_contratos.md`
