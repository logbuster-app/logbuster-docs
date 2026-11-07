# SPEC-FRONT-05 — Adaptar Busca de Filmes para Consumir o Back-end (`BACK-04` / `BACK-05`)

- **Status:** Approved
- **Card(s) Trello:** [`[FRONT-05]`](https://trello.com/c/rU1a52FF)
- **Repositório:** `logbuster-web`
- **Responsável:** Marcos Vinicius
- **Dependências:** [`SPEC-FRONT-01`](SPEC-FRONT-01-busca-filmes-omdb.md), [`SPEC-BACK-04`](../back/SPEC-BACK-04-get-busca-filmes-omdb.md), [`SPEC-BACK-05`](../back/SPEC-BACK-05-nota-media-filmes.md)

---

## 1. Contexto e Objetivo
Substituir na tela inicial de busca (`MovieSearch`) a consulta provisória feita diretamente pelo navegador à OMDb pelo consumo do endpoint `GET /api/v1/movies` do nosso Back-end (`BACK-04` + `BACK-05`), passando a exibir também no `MovieCard` a **nota média do filme no LogBuster** quando houver avaliações cadastradas e **atualizando obrigatoriamente a documentação do `logbuster-web`** para refletir a nova arquitetura.

> **Pré-preparo antes do Back finalizar:** A adaptação visual do `MovieCard` (exibição de `ratingAvg` / `ratingCount`) e o mapper `toMovieSummaryFromApi` com testes mockando `fetch` no contrato JSON abaixo podem ser adiantados imediatamente; o apontamento ativo em `useMovieSearch` deve ser mergeado quando `BACK-04` e `BACK-05` estiverem prontos (ou chaveado via `env` durante homologação).

## 2. Escopo
- **Inclui:** Novo cliente `searchMoviesFromApi` em `src/features/movies/api/` apontando para `${env.apiBaseUrl}/api/v1/movies`, extensão opcional de `MovieSummary` com `ratingAvg` e `ratingCount`, exibição discreta da nota média (`text-secondary`) no `MovieCard` quando `ratingCount > 0` e atualização das páginas do Docsify em `logbuster-web/docs/`.
- **Não inclui:** Ordenação geral de catálogo por popularidade/ano/nota (item registrado no [Radar Pós-Parcial](../../sdd/escopo-parcial-e-radar.md#radar-01--ordenacao-geral-de-filmes-mais-novos-mais-populares-mais-bem-avaliados)).

## 3. Regras de Negócio
- **RN-01 (Remoção da Chave OMDb no Fluxo de Busca do Navegador):** Após esta task, a busca de filmes no Front não exige `NEXT_PUBLIC_OMDB_API_KEY` no navegador; toda credencial da OMDb fica protegida no `logbuster-api`.
- **RN-02 (Exibição da Nota Média na Grade):**
  - Se `movie.ratingAvg !== null` e `movie.ratingCount > 0`, o `MovieCard` exibe a estrela âmbar (`text-secondary`) com a média formatada com 1 casa decimal (ex.: `★ 4,5`).
  - Se `movie.ratingAvg === null` (`ratingCount === 0`), o card exibe apenas título e ano (ou indicador discreto `"Sem nota"`), sem mostrar `0,0`.
- **RN-03 (Atualização Obrigatória de Documentação na Virada):** Como esta task encerra o caráter provisório da busca direta na OMDb, o PR **deve** atualizar `logbuster-web/docs/README.md`, `docs/setup.md`, `docs/integracao-omdb.md` (ou substituí-lo por `integracao-api.md`) e `docs/features/busca-de-filmes.md`.

## 4. Contrato Técnico (`src/features/movies`)

### 4.1 Chamada Consumida (`BACK-04` + `BACK-05`)
`GET {NEXT_PUBLIC_API_BASE_URL}/api/v1/movies?q={query}&page={page}`

Resposta `200 OK` esperada do Back-end:
```json
{
  "items": [
    {
      "id": "tt0133093",
      "title": "The Matrix",
      "year": "1999",
      "poster_url": "https://m.media-amazon.com/...",
      "rating_avg": 4.5,
      "rating_count": 3
    }
  ],
  "total": 142,
  "page": 1,
  "page_size": 10
}
```

### 4.2 Atualização de `MovieSummary` (`src/features/movies/types.ts`)
```ts
export type MovieSummary = {
  id: string;
  title: string;
  year: string;
  posterUrl: string | null;
  ratingAvg?: number | null;
  ratingCount?: number;
};
```

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Busca passa pelo `logbuster-api` e exibe nota média
- **Dado** que o backend retorna `"The Matrix"` com `rating_avg: 4.5` e `rating_count: 3`
- **Quando** o usuário busca `"Matrix"` na tela inicial
- **Então** o Front consulta `GET /api/v1/movies?q=Matrix&page=1` e o card renderiza pôster, título, ano e a nota `4,5` em destaque âmbar (`text-secondary`).

### Cenário 2: Filme sem avaliações não exibe nota zero enganosa
- **Dado** que um filme retornado pelo backend possui `rating_avg: null` e `rating_count: 0`
- **Quando** o card é renderizado na grade
- **Então** nenhuma nota `0` ou `0,0` é mostrada ao usuário.

### Cenário 3: Erro padronizado do backend é apresentado com segurança
- **Dado** que o backend retorna `502` com envelope `{ error: { code: "UPSTREAM_ERROR", message: "..." } }`
- **Quando** a busca é executada
- **Então** o mapper lança `ApiError` com a mensagem do envelope e `MovieSearch` renderiza `StateMessage tone="danger"`.

## 6. Definição de Pronto (DoD)
- [ ] `useMovieSearch` consumindo o cliente do `logbuster-api`.
- [ ] `MovieCard` exibindo `ratingAvg` quando disponível.
- [ ] Testes atualizados cobrindo o novo contrato e tratamento do envelope de erro (CI verde).
- [ ] **Docsify do `logbuster-web` atualizado** documentando a migração da OMDb do Front para o Back.
- [ ] Card `[FRONT-05]` atualizado com link para esta SPEC.
