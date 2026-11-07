# SPEC-BACK-05 — Endpoint 5: Nota Média dos Filmes (Enriquecimento de `GET Filmes` e Consulta por ID)

- **Status:** Approved
- **Card(s) Trello:** [`[BACK-05]`](https://trello.com/c/yWVNSm4E)
- **Repositório:** `logbuster-api`
- **Responsável:** Marcelo Palmeira
- **Dependências:** [`SPEC-BACK-01`](SPEC-BACK-01-post-cadastro-avaliacao.md), [`SPEC-BACK-04`](SPEC-BACK-04-get-busca-filmes-omdb.md)

---

## 1. Contexto e Objetivo
Calcular a **nota média oficial do LogBuster** (`rating_avg`) e o **total de avaliações** (`rating_count`) de cada filme a partir das avaliações salvas no banco de dados, enriquecendo a resposta de `GET /api/v1/movies` (`BACK-04`), servindo à ficha do filme em `GET /api/v1/movies/{movie_id}` (`BACK-07`) e expondo também `GET /api/v1/movies/{movie_id}/rating` para consulta leve/independente.

## 2. Escopo
- **Inclui:** Função agregadora em lote (`get_ratings_map(movie_ids)`) em uma única query `GROUP BY` (evitando problema N+1), inclusão de `rating_avg` e `rating_count` nos itens de `GET /api/v1/movies` e rota auxiliar `GET /api/v1/movies/{movie_id}/rating`.
- **Não inclui:** Ponderação por IA ou importação da nota do IMDb como se fosse nota da comunidade LogBuster.

## 3. Regras de Negócio
- **RN-01 (Cálculo da Média):** `rating_avg = round(AVG(rating), 2)` considerando todas as avaliações cadastradas para aquele `movie_id` no LogBuster.
- **RN-02 (Filme sem Avaliações):** Quando `rating_count == 0`, `rating_avg` **deve ser `null`** (nunca `0.0`), diferenciando ausência de notas de nota baixa.
- **RN-03 (Sem N+1 na Busca):** Ao enriquecer os até 10 filmes de `GET /api/v1/movies`, o backend realiza **apenas 1 consulta SQL** (`WHERE movie_id IN (...) GROUP BY movie_id`).

## 4. Contrato Técnico

### 4.1 Enriquecimento em `GET /api/v1/movies` (`BACK-04`)
Cada item da lista `items` passa a conter valores reais calculados do banco:
```json
{
  "id": "tt0133093",
  "title": "The Matrix",
  "year": "1999",
  "poster_url": "https://...",
  "rating_avg": 4.25,
  "rating_count": 4
}
```

### 4.2 Rota Direta `GET /api/v1/movies/{movie_id}/rating`
- **Response `200 OK` (com avaliações):**
  ```json
  {
    "movie_id": "tt0133093",
    "rating_avg": 4.25,
    "rating_count": 4
  }
  ```
- **Response `200 OK` (sem avaliações):**
  ```json
  {
    "movie_id": "tt0133093",
    "rating_avg": null,
    "rating_count": 0
  }
  ```

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Busca de filmes traz média real das avaliações cadastradas
- **Dado** que o filme `tt0133093` possui duas avaliações com notas `4.0` e `4.5` e o filme `tt0234215` não possui avaliações
- **Quando** `GET /api/v1/movies?q=Matrix` retorna ambos os títulos
- **Então** `tt0133093` apresenta `rating_avg: 4.25, rating_count: 2` e `tt0234215` apresenta `rating_avg: null, rating_count: 0`.

### Cenário 2: Consulta direta de nota média por `movie_id`
- **Dado** qualquer `movie_id` válido (`^tt\d{7,10}$`)
- **Quando** `GET /api/v1/movies/{movie_id}/rating` é chamado
- **Então** retorna HTTP `200` com `rating_avg` e `rating_count` atualizados.

## 6. Definição de Pronto (DoD)
- [ ] Agregação SQL em lote integrada ao `GET /api/v1/movies` e rota `GET /api/v1/movies/{movie_id}/rating` implementada.
- [ ] Testes cobrindo filme sem avaliações, filme com múltiplas avaliações e recálculo após novo `POST /api/v1/reviews` (CI verde).
- [ ] Documentação atualizada em `logbuster-api/docs/`.
- [ ] Card `[BACK-05]` atualizado com descrição e link para esta SPEC.
