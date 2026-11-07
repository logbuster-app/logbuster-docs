# SPEC-BACK-07 — Endpoint 7: `GET` Consulta de Filme Específico na OMDb + Nota Média

- **Status:** Approved
- **Card(s) Trello:** [`[BACK-07]`](https://trello.com/c/BKdWBAp8)
- **Repositório:** `logbuster-api`
- **Responsável:** Marcelo Palmeira
- **Dependências:** [`SPEC-BACK-04`](SPEC-BACK-04-get-busca-filmes-omdb.md), [`SPEC-BACK-05`](SPEC-BACK-05-nota-media-filmes.md)

---

## 1. Contexto e Objetivo
Expor no Back-end a ficha detalhada de um filme específico (`GET /api/v1/movies/{movie_id}`), consultando a OMDb por `imdbID` (`i={movie_id}&plot=full`), normalizando todos os metadados e agregando na mesma resposta a **nota média oficial do LogBuster** (`rating_avg` e `rating_count`) para consumo direto em `FRONT-06`.

## 2. Escopo
- **Inclui:** Método `get_movie_by_id(movie_id)` no `OmdbClient`, normalização de campos `"N/A"` e lista `genres`, agregação com o serviço de média de `BACK-05` e rota `GET /api/v1/movies/{movie_id}`.
- **Não inclui:** Filmografia detalhada de pessoas, temporadas de séries ou resumo por IA.

## 3. Regras de Negócio
- **RN-01 (Validação de Formato):** `movie_id` fora do padrão `^tt\d{7,10}$` retorna `422 VALIDATION_ERROR` antes mesmo de consumir requisição na OMDb.
- **RN-02 (Filme Não Encontrado na OMDb):** Quando a OMDb retorna `Response: "False"` com `"Incorrect IMDb ID."` ou `"Movie not found!"`, o endpoint retorna HTTP `404 NOT_FOUND` no envelope padrão.
- **RN-03 (Normalização de `"N/A"` e `Genre`):**
  - Campos textuais (`Year`, `Runtime`, `Director`, `Actors`, `Plot`, `Poster`) iguais a `"N/A"` ou vazios são convertidos para `null`.
  - `Genre` (ex.: `"Action, Sci-Fi"`) é separado por vírgula em lista `["Action", "Sci-Fi"]` (ou `[]` se `"N/A"`).
- **RN-04 (Nota Média Integrada):** Anexa `rating_avg` (`float | null`) e `rating_count` (`int`) vindos do banco do LogBuster.

## 4. Contrato Técnico

- **Método e Rota:** `GET /api/v1/movies/{movie_id}`
- **Response `200 OK`:**
  ```json
  {
    "id": "tt0133093",
    "title": "The Matrix",
    "year": "1999",
    "runtime": "136 min",
    "genres": ["Action", "Sci-Fi"],
    "director": "Lana Wachowski, Lilly Wachowski",
    "actors": "Keanu Reeves, Laurence Fishburne, Carrie-Anne Moss",
    "plot": "When a beautiful stranger leads computer hacker Neo to a forbidding underworld...",
    "poster_url": "https://m.media-amazon.com/images/M/MV5BNzQzOTk3OTAtNDQ0Zi00ZTVkLWI0MTEtMDllZjNkYzNjNTc4L2ltYWdlXkEyXkFqcGdeQXVyNjU0OTQ0OTY@._V1_SX300.jpg",
    "rating_avg": 4.5,
    "rating_count": 12
  }
  ```
- **Erros:** `404 NOT_FOUND`, `422 VALIDATION_ERROR`, `502 UPSTREAM_ERROR`.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Retorno completo combinando OMDb e nota média do LogBuster
- **Dado** que a OMDb encontra `tt0133093` e há 2 avaliações no banco com média `4.5`
- **Quando** `GET /api/v1/movies/tt0133093` é requisitado
- **Então** retorna HTTP `200` com metadados normalizados, `genres` em array e `rating_avg: 4.5, rating_count: 2`.

### Cenário 2: Filme inexistente na OMDb
- **Dado** que a OMDb retorna `{"Response": "False", "Error": "Incorrect IMDb ID."}`
- **Quando** `GET /api/v1/movies/tt0000000` é requisitado
- **Então** retorna HTTP `404` com `error.code == "NOT_FOUND"`.

## 6. Definição de Pronto (DoD)
- [ ] Rota `GET /api/v1/movies/{movie_id}` implementada reutilizando `OmdbClient` e cálculo de média.
- [ ] Testes cobrindo `200` (com e sem `"N/A"`), `404`, `422` e `502` passando no CI.
- [ ] Documentação atualizada em `logbuster-api/docs/`.
- [ ] Card `[BACK-07]` atualizado com responsável, descrição e link para esta SPEC.
