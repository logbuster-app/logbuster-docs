# SPEC-FRONT-06 — Adaptar Tela de Filme Específico para Consumir o Back-end (`BACK-07`)

- **Status:** Approved
- **Card(s) Trello:** [`[FRONT-06]`](https://trello.com/c/MMFGf9pn)
- **Repositório:** `logbuster-web`
- **Responsável:** Marcos Vinicius
- **Dependências:** [`SPEC-FRONT-02`](SPEC-FRONT-02-tela-filme-especifico.md), [`SPEC-BACK-07`](../back/SPEC-BACK-07-get-filme-especifico-omdb.md)

---

## 1. Contexto e Objetivo
Substituir na tela de detalhe do filme (`/movies/[id]`) a consulta provisória direta à OMDb pela chamada ao endpoint `GET /api/v1/movies/{movie_id}` do nosso Back-end (`BACK-07`), recebendo em uma única requisição os metadados normalizados do filme e a **nota média oficial calculada pelo LogBuster** (`rating_avg` e `rating_count`), além de atualizar a documentação local do `logbuster-web`.

> **Pré-preparo antes do Back finalizar:** O cliente `getMovieByIdFromApi` e seus testes unitários com resposta mockada no contrato de `BACK-07` podem ser implementados antecipadamente; a troca definitiva da fonte padrão na rota `/movies/[id]` entra assim que `BACK-07` estiver validado.

## 2. Escopo
- **Inclui:** Adaptar `getMovieById` em `src/features/movies/api/` para consumir `GET {NEXT_PUBLIC_API_BASE_URL}/api/v1/movies/{id}`, mapear `rating_avg` e `rating_count` para `MovieDetail`, permitir revalidar os dados do filme quando uma nova avaliação for criada em `FRONT-04` (atualizando a nota média em tela) e atualizar o Docsify de `logbuster-web`.
- **Não inclui:** Alterações de layout fora do que já foi definido em `SPEC-FRONT-02` e `SPEC-UI-01`.

## 3. Regras de Negócio
- **RN-01 (Preservação do Tipo de Domínio `MovieDetail`):** A troca acontece na camada `features/movies/api/`; o componente `MovieDetail` continua recebendo o mesmo contrato TypeScript `MovieDetail`.
- **RN-02 (Atualização Reativa da Nota Média):** Quando o usuário publica uma nova avaliação pelo modal (`FRONT-04`), a tela `/movies/[id]` refaz a leitura de `getMovieById` (ou atualiza o estado) para refletir imediatamente a nova `ratingAvg` e `ratingCount`.
- **RN-03 (Remoção Definitiva de Dependência OMDb no Front):** Concluídos `FRONT-05` e `FRONT-06`, o `logbuster-web` não faz mais nenhuma chamada direta a `omdbapi.com`. As variáveis e documentos de setup do Front passam a exigir apenas `NEXT_PUBLIC_API_BASE_URL`.

## 4. Contrato Técnico (`src/features/movies`)

### 4.1 Endpoint Consumido (`BACK-07`)
`GET {NEXT_PUBLIC_API_BASE_URL}/api/v1/movies/{movie_id}`

Resposta `200 OK`:
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
  "poster_url": "https://m.media-amazon.com/...",
  "rating_avg": 4.5,
  "rating_count": 12
}
```
- `404 NOT_FOUND` → mapeado para estado `not_found` (`null`).
- `502 UPSTREAM_ERROR` / rede → lança `ApiError`.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Carregar detalhes e nota média vindos do nosso Back-end
- **Dado** que o usuário acessa `/movies/tt0133093`
- **Quando** `GET /api/v1/movies/tt0133093` retorna `200 OK` com `rating_avg: 4.5` e `rating_count: 12`
- **Então** a ficha exibe os metadados do filme e destaca a média `4,5` (`12 avaliações`) sem nenhuma requisição direta à OMDb no navegador.

### Cenário 2: Filme inexistente retorna 404 do Back-end
- **Dado** que o usuário acessa `/movies/tt9999999`
- **Quando** o backend responde `404` com `{ error: { code: "NOT_FOUND", ... } }`
- **Então** a tela renderiza a mensagem amigável de filme não encontrado.

## 6. Definição de Pronto (DoD)
- [ ] `getMovieById` migrado para `GET /api/v1/movies/{id}` com mapper e revalidação pós-avaliação.
- [ ] Testes unitários e de componente cobrindo `200`, `404` e `502` passando no CI.
- [ ] Documentação em `logbuster-web/docs/features/detalhe-de-filme.md` e `docs/setup.md` atualizada.
- [ ] Card `[FRONT-06]` preenchido e vinculado a esta SPEC.
