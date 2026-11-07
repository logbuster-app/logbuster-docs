# SPEC-BACK-04 — Endpoint 4: `GET` Busca de Filmes com Integração à OMDb

- **Status:** Approved
- **Card(s) Trello:** [`[BACK-04]`](https://trello.com/c/NfQxCkHy)
- **Repositório:** `logbuster-api`
- **Responsável:** Marcelo Palmeira
- **Dependências:** [`SPEC-BACK-00`](SPEC-BACK-00-setup-arquitetura-api.md), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md)

---

## 1. Contexto e Objetivo
Centralizar no Back-end (`logbuster-api`) a busca textual de filmes na API externa **OMDb**, encapsulando a chave `OMDB_API_KEY` no servidor, normalizando o formato cru da OMDb para o contrato limpo do LogBuster e servindo de base para o enriquecimento com nota média (`BACK-05`) e consumo pelo Front-end (`FRONT-05`).

## 2. Escopo
- **Inclui:** Cliente HTTP isolado `OmdbClient` (`app/modules/movies/omdb_client.py`), rota `GET /api/v1/movies?q=...&page=...`, filtro fixo `type=movie` na OMDb, normalização de `Poster: "N/A"` para `null` e tratamento de erros da OMDb.
- **Não inclui:** Busca de séries ou ordenação geral de catálogo (registrada no [Radar Pós-Parcial](../../sdd/escopo-parcial-e-radar.md#radar-01--ordenacao-geral-de-filmes-mais-novos-mais-populares-mais-bem-avaliados)).

## 3. Regras de Negócio
- **RN-01 (Validação de `q`):** O query param `q` é obrigatório e não pode ser vazio após `strip()` (`422 VALIDATION_ERROR`).
- **RN-02 (Tratamento das Respostas Peculiares da OMDb):** A OMDb frequentemente responde HTTP `200` com `{"Response": "False", "Error": "..."}`. O `OmdbClient` deve traduzir:
  - `"Movie not found!"` → **Não é erro:** retorna `200 OK` com `items: []`, `total: 0`.
  - `"Too many results."` → retorna `422 VALIDATION_ERROR` com mensagem `"Muitos resultados. Digite um termo de busca mais específico."`.
  - `"Invalid API key!"`, HTTP `401`, timeout ou erro `5xx` da OMDb → retorna `502 UPSTREAM_ERROR` sem expor a chave de API nos detalhes.
- **RN-03 (Normalização de Campos):** `Poster == "N/A"` ou vazio torna-se `poster_url: null`; `totalResults` (string na OMDb) torna-se inteiro `total`; `page_size` fixo em `10` (padrão da OMDb).

## 4. Contrato Técnico

- **Método e Rota:** `GET /api/v1/movies`
- **Query Params:**
  - `q`: string obrigatória (ex.: `"Matrix"`)
  - `page`: inteiro `>= 1` (default `1`)
- **Response `200 OK` *(já prevendo os campos enriquecidos em `BACK-05`)*:**
  ```json
  {
    "items": [
      {
        "id": "tt0133093",
        "title": "The Matrix",
        "year": "1999",
        "poster_url": "https://m.media-amazon.com/images/M/MV5BNzQzOTk3OTAtNDQ0Zi00ZTVkLWI0MTEtMDllZjNkYzNjNTc4L2ltYWdlXkEyXkFqcGdeQXVyNjU0OTQ0OTY@._V1_SX300.jpg",
        "rating_avg": null,
        "rating_count": 0
      }
    ],
    "total": 142,
    "page": 1,
    "page_size": 10
  }
  ```

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Busca por texto retorna filmes normalizados
- **Dado** que a OMDb retorna itens em `Search` sendo um deles com `Poster: "N/A"`
- **Quando** `GET /api/v1/movies?q=Matrix&page=1` é chamado
- **Então** retorna HTTP `200` com `items` mapeados (`id`, `title`, `year`, `poster_url: null` para `"N/A"`) e `total` numérico.

### Cenário 2: Nenhum filme encontrado na OMDb
- **Dado** que a OMDb retorna `{"Response": "False", "Error": "Movie not found!"}`
- **Quando** `GET /api/v1/movies?q=termoinexistente` é chamado
- **Então** retorna HTTP `200` com `items: []` e `total: 0`.

### Cenário 3: Timeout ou falha na OMDb
- **Dado** que a chamada externa à OMDb sofre timeout ou retorna `401`
- **Quando** `GET /api/v1/movies?q=Matrix` é chamado
- **Então** retorna HTTP `502` com `error.code == "UPSTREAM_ERROR"`.

## 6. Definição de Pronto (DoD)
- [ ] `OmdbClient` isolado e mockável + rota `GET /api/v1/movies` implementada.
- [ ] Testes automatizados com mocks de `httpx` cobrindo sucesso, `"N/A"`, `"Movie not found!"`, `"Too many results."` e falha `502` (CI verde).
- [ ] Documentação atualizada em `logbuster-api/docs/`.
- [ ] Card `[BACK-04]` atualizado com descrição e link para esta SPEC.
