# SPEC-BACK-02 — Endpoint 2: `GET` Leitura e Ordenação de Avaliações por Filme

- **Status:** Approved
- **Card(s) Trello:** [`[BACK-02]`](https://trello.com/c/QgTm932N)
- **Repositório:** `logbuster-api`
- **Responsável:** João Victor
- **Dependências:** [`SPEC-BACK-01`](SPEC-BACK-01-post-cadastro-avaliacao.md), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md)

---

## 1. Contexto e Objetivo
Retornar a lista paginada de avaliações de um filme específico (`movie_id`), ordenando **por padrão pela quantidade de votos `"Bem argumentada"` (`well_argued`)** — diferencial central do LogBuster — e suportando ordenações alternativas solicitadas pelo usuário na tela do filme (`FRONT-03`).

## 2. Escopo
- **Inclui:** Rota `GET /api/v1/movies/{movie_id}/reviews`, parâmetros de paginação (`page`, `page_size`) e ordenação (`sort`), agregação dos contadores `like`, `spoiler` e `well_argued` e preenchimento opcional de `viewer_interactions` quando o header `X-Visitor-Id` for informado.
- **Não inclui:** Resumo por IA ou filtragem por perfil privado.

## 3. Regras de Negócio
- **RN-01 (Ordenações Suportadas em `sort`):**
  - `"well_argued"` **(padrão quando omitido):** ordena por `counts.well_argued DESC`, desempate por `counts.like DESC` e depois `created_at DESC`.
  - `"most_liked"`: ordena por `counts.like DESC`, desempate por `counts.well_argued DESC` e `created_at DESC`.
  - `"newest"`: ordena por `created_at DESC`.
  - `"rating_desc"`: ordena por `rating DESC`, desempate por `created_at DESC`.
  - `"rating_asc"`: ordena por `rating ASC`, desempate por `created_at DESC`.
- **RN-02 (Filme sem Avaliações):** Se não houver nenhuma avaliação para o `movie_id` válido informado, o endpoint retorna `200 OK` com `items: []` e `total: 0` (nunca `404`).
- **RN-03 (Interações do Visitante Atual):** Se o header `X-Visitor-Id` estiver presente, `viewer_interactions` traz a lista de tipos (`["like", "well_argued"]`) que aquele visitante já marcou em cada avaliação; se ausente, retorna `[]`.

## 4. Contrato Técnico

- **Método e Rota:** `GET /api/v1/movies/{movie_id}/reviews`
- **Headers Opcionais:** `X-Visitor-Id: <uuid>`
- **Query Params:**
  - `sort`: `well_argued` (default) | `most_liked` | `newest` | `rating_desc` | `rating_asc`
  - `page`: inteiro `>= 1` (default `1`)
  - `page_size`: inteiro `1..50` (default `10`)
- **Response `200 OK`:**
  ```json
  {
    "items": [
      {
        "id": "c4f7d9d2-9d4e-4b3a-8f55-1f2e3d4c5b6a",
        "movie_id": "tt0133093",
        "author_name": "Thyago",
        "rating": 4.5,
        "content": "Direção precisa e efeitos visuais que revolucionaram o gênero.",
        "is_spoiler": false,
        "highlights": ["direction", "visual_effects"],
        "counts": {
          "like": 4,
          "spoiler": 0,
          "well_argued": 9
        },
        "viewer_interactions": ["well_argued"],
        "created_at": "2026-11-08T18:30:00Z"
      }
    ],
    "total": 1,
    "page": 1,
    "page_size": 10,
    "sort": "well_argued"
  }
  ```

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Ordenação padrão destaca a análise mais bem argumentada
- **Dado** um filme com 3 avaliações tendo `well_argued` iguais a `1`, `8` e `3`
- **Quando** `GET /api/v1/movies/tt0133093/reviews` é chamado sem parâmetro `sort`
- **Então** retorna HTTP `200` com as avaliações na ordem `8`, `3`, `1`.

### Cenário 2: Ordenações alternativas e lista vazia
- **Dado** chamadas com `sort=newest`, `sort=most_liked`, `sort=rating_desc` e para um filme sem avaliações
- **Quando** processadas pela API
- **Então** respeitam cada critério de ordenação e retornam `items: [], total: 0` quando não há registros.

## 6. Definição de Pronto (DoD)
- [ ] Rota `GET /api/v1/movies/{movie_id}/reviews` implementada com ordenações e paginação.
- [ ] Testes automatizados cobrindo todas as opções de `sort`, desempate, paginação e `viewer_interactions` (CI verde).
- [ ] Documentação atualizada em `logbuster-api/docs/`.
- [ ] Card `[BACK-02]` atualizado com descrição e link para esta SPEC.
