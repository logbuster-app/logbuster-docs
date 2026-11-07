# SPEC-FRONT-02 — Tela de Visualização de Filme Específico (Fase Provisória OMDb)

- **Status:** Approved
- **Card(s) Trello:** [`[FRONT-02]`](https://trello.com/c/8PfLP55W)
- **Repositório:** `logbuster-web`
- **Responsável:** Elton Gonçalves
- **Dependências:** [`SPEC-FRONT-01`](SPEC-FRONT-01-busca-filmes-omdb.md), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md)

---

## 1. Contexto e Objetivo
Permitir que o usuário clique em qualquer filme listado na busca (`MovieCard`) e abra a página dedicada do filme (`/movies/[id]`), visualizando seus metadados completos vindos provisoriamente da OMDb no Front-end e deixando a estrutura pronta (slots de composição) para receber a nota média do LogBuster, o botão de avaliar (`FRONT-04`) e a seção de avaliações (`FRONT-03`).

> **Nota de evolução:** Quando o endpoint [`SPEC-BACK-07`](../back/SPEC-BACK-07-get-filme-especifico-omdb.md) estiver pronto, o card [`SPEC-FRONT-06`](SPEC-FRONT-06-adaptar-detalhe-filme-backend.md) trocará apenas a camada `api/` desta feature para consultar o nosso Back-end (incluindo a nota média), mantendo o componente visual intacto.

## 2. Escopo
- **Inclui:** Tornar `MovieCard` navegável para `/movies/[id]`, rota fina `src/app/movies/[id]/page.tsx`, função `getMovieById(id, signal?)` reutilizando `omdbRequest({ i: id, plot: "full" })`, mapper `toMovieDetail`, componente `MovieDetail` com estados `loading`, `error` e `notFound` e área reservada para avaliações.
- **Não inclui:** Filmografia clicável de atores/diretores, temporadas de séries ou resumo por IA.

## 3. Regras de Negócio
- **RN-01 (Navegação):** Clicar em um `MovieCard` na grade de busca navega para `/movies/<imdbID>` (ex.: `/movies/tt0133093`), mantendo link de retorno para a busca.
- **RN-02 (Normalização de Campos `"N/A"`):** Na resposta de detalhe da OMDb, qualquer campo igual a `"N/A"` ou ausente (`Poster`, `Plot`, `Director`, `Actors`, `Runtime`, `Genre`, `Year`) deve ser convertido para `null` (ou `[]` em `genres`), exibindo fallback discreto na UI (ex.: `"Sinopse não informada"`) sem mostrar a string literal `"N/A"`.
- **RN-03 (Filme Não Encontrado):** Se a OMDb responder `Response: "False"` (ex.: `"Incorrect IMDb ID."` ou `"Movie not found!"`), a tela exibe estado amigável de filme não encontrado com ação para voltar à busca.
- **RN-04 (Desacoplamento entre Features):** Seguindo a regra arquitetural de `logbuster-web`, `features/movies` **não importa** de `features/reviews`. A composição entre `MovieDetail` e a seção de avaliações (`FRONT-03` / `FRONT-04`) deve ocorrer na rota `src/app/movies/[id]/page.tsx` ou via slot `children` / props de renderização.

## 4. Contrato Técnico (`src/features/movies`)

### 4.1 Modelo de Domínio (`src/features/movies/types.ts`)
```ts
export type MovieDetail = {
  id: string; // imdbID, ex.: "tt0133093"
  title: string;
  year: string | null;
  runtime: string | null;
  genres: string[];
  director: string | null;
  actors: string | null;
  plot: string | null;
  posterUrl: string | null;
  /** Preenchido quando integrado ao backend (FRONT-06 / BACK-05); null enquanto provisório ou sem notas. */
  ratingAvg: number | null;
  ratingCount: number;
};
```

### 4.2 Arquivos Esperados
- `src/features/movies/api/omdb.ts`: adicionar `getMovieById(id: string, signal?: AbortSignal): Promise<MovieDetail | null>` e `toMovieDetail`.
- `src/features/movies/hooks/useMovieDetail.ts`: gerenciar estados `loading | success | not_found | error`.
- `src/features/movies/components/MovieCard.tsx`: envolver card com `Link` acessível para `/movies/${movie.id}`.
- `src/features/movies/components/MovieDetail.tsx`: exibir pôster `aspect-[2/3]`, título, ano, duração, gêneros em badges, direção, elenco, sinopse, indicador de nota LogBuster (`text-secondary`, mostrando `"Ainda sem avaliações"` quando `ratingCount === 0`) e área inferior para avaliações.
- `src/app/movies/[id]/page.tsx`: rota fina compondo `MovieDetail`.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Abrir ficha de filme a partir da busca
- **Dado** que a busca listou o filme `"The Matrix"` (`id: "tt0133093"`)
- **Quando** o usuário clica no card do filme
- **Então** a aplicação navega para `/movies/tt0133093`, exibe indicador de carregamento e, ao concluir, renderiza pôster, título, ano, duração, gêneros, direção, elenco e sinopse usando apenas tokens de [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md).

### Cenário 2: ID de filme inexistente
- **Dado** que o usuário acessa `/movies/tt0000000`
- **Quando** a OMDb retorna `Response: "False"`
- **Então** a tela exibe `StateMessage` informando que o filme não foi encontrado e um link para voltar à busca.

### Cenário 3: Metadados parciais (`"N/A"`)
- **Dado** um filme na OMDb com `Poster: "N/A"` e `Plot: "N/A"`
- **Quando** a tela renderiza os detalhes
- **Então** o pôster mostra o placeholder `"Sem poster"` e a literal `"N/A"` não aparece em nenhum ponto da interface.

## 6. Definição de Pronto (DoD)
- [ ] Rota `/movies/[id]` e componente `MovieDetail` implementados e ligados ao `MovieCard`.
- [ ] Testes unitários/componentes cobrindo sucesso, `"N/A"`, não encontrado e erro de rede (CI passando com cobertura mínima).
- [ ] Página `logbuster-web/docs/features/detalhe-de-filme.md` criada e adicionada ao `_sidebar.md`.
- [ ] Card `[FRONT-02]` atualizado com link para esta SPEC.
