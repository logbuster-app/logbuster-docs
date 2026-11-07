# SPEC-FRONT-01 — Busca de Filmes por Texto (Integração Provisória OMDb no Front)

- **Status:** Implemented (As-Built — PR #1)
- **Card(s) Trello:** [`[FRONT-01]`](https://trello.com/c/2WAFMwRD) *(Revisão: Elton e Thyago)*
- **Repositório:** `logbuster-web`
- **Responsável:** Marcos Vinicius
- **Dependências:** [`SPEC-FRONT-00`](SPEC-FRONT-00-setup-arquitetura-web.md), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md)

---

## 1. Contexto e Objetivo
Entregar a primeira tela funcional do LogBuster (`/`), onde o usuário busca filmes por título e visualiza a grade de resultados consultando provisoriamente a API gratuita da OMDb direto pelo Front-end — isolada por mapper de domínio para futura troca transparente pelo Back-end em [`SPEC-FRONT-05`](SPEC-FRONT-05-adaptar-busca-para-backend.md).

## 2. Escopo
- **Inclui:** Barra de busca por formulário, persistência do termo na URL (`/?q=`), consulta `GET` à OMDb (`type=movie`), paginação incremental (*"Carregar mais"*, 10 itens por página), grade responsiva de pôsteres e tratamento de todos os estados visuais.
- **Não inclui:** Busca de séries, autocomplete a cada tecla (para preservar cota gratuita da OMDb) ou ordenação avançada de catálogo.

## 3. Regras de Negócio
- **RN-01 (Disparo Explícito):** A requisição só ocorre ao submeter o formulário (botão **Buscar** ou tecla `Enter`) com texto não vazio após `trim()`.
- **RN-02 (Estado na URL):** O termo pesquisado reflete em `/?q=<termo>`, permitindo recarregar, compartilhar link ou voltar da ficha do filme preservando os resultados.
- **RN-03 (Mapeamento Resiliente da OMDb):**
  - `Poster: "N/A"` é convertido para `posterUrl: null` e renderiza placeholder `"Sem poster"`.
  - `Error: "Movie not found!"` não é falha técnica: retorna lista vazia (`movies: [], total: 0`).
  - `Error: "Too many results."`, chave ausente/inválida (`401`) ou erro de rede lançam `ApiError` com mensagem amigável em português.

## 4. Contrato Técnico (`src/features/movies`)
- **Tipos de domínio (`types.ts`):**
  ```ts
  export type MovieSummary = {
    id: string; // imdbID, ex.: "tt0133093"
    title: string;
    year: string;
    posterUrl: string | null;
  };

  export type MovieSearchPage = {
    movies: MovieSummary[];
    total: number;
    page: number;
    pageSize: number;
  };
  ```
- **Arquivos implementados:** `api/omdb.ts`, `hooks/useMovieSearch.ts`, `components/{SearchBar,MovieCard,MovieGrid,MovieSearch}.tsx` e página `src/app/page.tsx` com `<Suspense>`.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Busca com resultados e paginação
- **Dado** que o usuário acessa `/` e digita `"Matrix"` clicando em **Buscar**
- **Quando** a OMDb retorna resultados válidos com `totalResults > 10`
- **Então** a URL muda para `/?q=Matrix`, a grade exibe até 10 cards com pôster/título/ano e o botão **"Carregar mais"** anexa a página 2 sem apagar a página 1.

### Cenário 2: Nenhum filme encontrado
- **Dado** que o usuário busca um termo inexistente na OMDb
- **Quando** a resposta traz `{ Response: "False", Error: "Movie not found!" }`
- **Então** a tela exibe `StateMessage` neutro informando `"Nenhum filme encontrado"`.

### Cenário 3: Falha de rede ou chave ausente
- **Dado** que `NEXT_PUBLIC_OMDB_API_KEY` está vazia ou a rede falha
- **Quando** o usuário realiza uma busca
- **Então** a tela exibe `StateMessage` com `tone="danger"` (`role="alert"`) sem quebrar a renderização.

## 6. Definição de Pronto (DoD)
- [x] Código implementado em `src/features/movies`.
- [x] Testes unitários/integração (`omdb.test.ts`, `SearchBar.test.tsx`, `MovieGrid.test.tsx`, `MovieSearch.test.tsx`) cobrindo sucesso, vazio e erro no CI.
- [x] Documentado em `logbuster-web/docs/features/busca-de-filmes.md` e `integracao-omdb.md`.
- [ ] Revisão final por Elton e Thyago e link desta SPEC no card `[FRONT-01]`.
