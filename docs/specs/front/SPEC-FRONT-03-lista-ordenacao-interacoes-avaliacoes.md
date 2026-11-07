# SPEC-FRONT-03 — Visualização, Ordenação e Interação com Avaliações

- **Status:** Approved
- **Card(s) Trello:** [`[FRONT-03]`](https://trello.com/c/pRQiSMGn)
- **Repositório:** `logbuster-web`
- **Responsável:** Elton Gonçalves
- **Dependências:** [`SPEC-FRONT-02`](SPEC-FRONT-02-tela-filme-especifico.md), [`SPEC-BACK-02`](../back/SPEC-BACK-02-get-lista-avaliacoes.md), [`SPEC-BACK-06`](../back/SPEC-BACK-06-post-interacao-avaliacao.md), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md)

---

## 1. Contexto e Objetivo
Exibir na tela do filme (`/movies/[id]`) as avaliações da comunidade vindas do nosso Back-end (`logbuster-api`), permitindo **ordená-las pela qualidade do argumento** (*"Bem argumentada"* como padrão, além de curtidas, data e nota) e **interagir com cada análise** por meio dos botões **`BEM ARGUMENTADA`**, **`LIKE`** e **`SPOILER`**, ocultando análises com spoiler até que o leitor decida revelá-las.

> **Estratégia de Desacoplamento do Back-end (Entrega em 2 PRs):**
> Para não bloquear o Front enquanto `BACK-02` e `BACK-06` estão em desenvolvimento:
> - **PR-1 (Mock-First — Independente do Back):** Implementar `ReviewList`, `ReviewCard`, seletor de ordenação e botões de interação consumindo um provider mockado em `src/features/reviews/api/reviews.mock.ts` (controlado por `NEXT_PUBLIC_USE_API_MOCKS=true` em `env.ts`), respeitando **exatamente** os tipos de domínio e regras de ordenação/toggle desta SPEC.
> - **PR-2 (Integração Real):** Conectar a camada `src/features/reviews/api/` aos endpoints `GET /api/v1/movies/{movie_id}/reviews` (`BACK-02`) e `POST /api/v1/reviews/{review_id}/interactions` (`BACK-06`), desligando o mock por padrão sem precisar alterar os componentes `.tsx`.

## 2. Escopo
- **Inclui:** Cliente HTTP de leitura/interação em `src/features/reviews/api`, geração/leitura de `X-Visitor-Id` em `src/shared/lib/visitor.ts`, seletor de ordenação de avaliações, lista de cards de avaliação (autor, data, estrelas `0,5–5,0`, badges de pontos de destaque, texto com proteção de spoiler) e botões de toggle para `like`, `spoiler` e `well_argued`.
- **Não inclui:** `DISLIKE`, comentários aninhados (threads) em avaliações, resumo de consenso por IA ou tag de spoiler inline no meio de frases.

## 3. Regras de Negócio
- **RN-01 (Ordenação Padrão por Argumento):** Ao carregar a seção de avaliações, a ordenação selecionada por padrão deve ser **`well_argued`** (*Mais bem argumentadas*), materializando o diferencial central do produto. O usuário pode alternar o seletor entre:
  - `well_argued` — *Mais bem argumentadas* **(padrão)**
  - `most_liked` — *Mais curtidas*
  - `newest` — *Mais recentes*
  - `rating_desc` — *Maior nota*
  - `rating_asc` — *Menor nota*
- **RN-02 (Três Interações Válidas com Toggle):** Cada avaliação exibe os botões **Bem argumentada** (`well_argued`), **Curtir** (`like`) e **Spoiler** (`spoiler`) com seus respectivos contadores. Clicar envia `POST /api/v1/reviews/{id}/interactions` com header `X-Visitor-Id`: se ainda não estava marcado pelo visitante, marca e incrementa; se já estava marcado, desmarca e decrementa.
- **RN-03 (Exibição Segura de Spoiler):** Quando `review.isSpoiler === true` **ou** `review.counts.spoiler >= 3`, o texto da avaliação inicia oculto por um bloco de alerta (`"Esta avaliação contém spoiler"`) com botão **"Mostrar análise"**. Revelar o texto altera apenas o estado local daquele card no navegador, sem remover a marcação no backend.
- **RN-04 (Estados Obrigatórios):** A seção trata explicitamente `loading` (`Spinner`), `empty` (`"Nenhuma avaliação ainda. Seja o primeiro a avaliar!"`), `error` (`StateMessage tone="danger"`) e falha pontual ao clicar em um voto.

## 4. Contrato Técnico (`src/features/reviews`)

### 4.1 Tipos de Domínio (`src/features/reviews/types.ts`)
```ts
export type ReviewId = string;

export type ReviewInteraction = "like" | "spoiler" | "well_argued";

export type ReviewHighlight =
  | "soundtrack"
  | "cinematography"
  | "screenplay"
  | "acting"
  | "direction"
  | "visual_effects";

export type ReviewSort =
  | "well_argued"
  | "most_liked"
  | "newest"
  | "rating_desc"
  | "rating_asc";

export type Review = {
  id: ReviewId;
  movieId: string;
  authorName: string;
  rating: number; // 0.5 a 5.0
  content: string;
  isSpoiler: boolean;
  highlights: ReviewHighlight[];
  counts: {
    like: number;
    spoiler: number;
    wellArgued: number;
  };
  viewerInteractions: ReviewInteraction[];
  createdAt: string;
};
```

### 4.2 Rótulos em Português dos Destaques (UI)
- `soundtrack` → `"Trilha Sonora"`
- `cinematography` → `"Fotografia"`
- `screenplay` → `"Roteiro"`
- `acting` → `"Atuação"`
- `direction` → `"Direção"`
- `visual_effects` → `"Efeitos Especiais"`

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Carregamento ordenado por "Mais bem argumentadas"
- **Dado** que o usuário abre `/movies/tt0133093`
- **Quando** a lista de avaliações é consultada no backend sem filtro manual prévio
- **Então** a requisição usa `sort=well_argued` e renderiza cada card com autor, nota em estrelas, chips de destaques e contadores de `Bem argumentada`, `Curtir` e `Spoiler`.

### Cenário 2: Alternar ordenação das avaliações
- **Dado** a lista de avaliações exibida na tela do filme
- **Quando** o usuário muda o seletor de ordenação para `"Mais recentes"` (`newest`) ou `"Maior nota"` (`rating_desc`)
- **Então** uma nova consulta `GET` é disparada com o parâmetro `sort` selecionado e a lista é reordenada conforme resposta do backend.

### Cenário 3: Ocultação e revelação local de spoiler
- **Dado** uma avaliação retornada com `is_spoiler: true` (ou `counts.spoiler >= 3`)
- **Quando** o card aparece na tela
- **Então** o texto da avaliação não fica legível inicialmente e, ao clicar em **"Mostrar análise"**, apenas aquela avaliação passa a exibir seu conteúdo textual.

### Cenário 4: Voto qualificado e toggle
- **Dado** uma avaliação em que o visitante ainda não clicou em **"Bem argumentada"**
- **Quando** o visitante clica no botão **"Bem argumentada"**
- **Então** o front envia `POST /api/v1/reviews/{id}/interactions` com `{ "type": "well_argued" }` e header `X-Visitor-Id`, atualizando o contador e destacando visualmente o botão como ativo.

## 6. Definição de Pronto (DoD)
- [ ] **PR-1 (Mock-First):** `ReviewList`, `ReviewCard`, seletor de ordenação, ocultação de spoiler e toggle de votos funcionando sobre `reviews.mock.ts` com testes passando no CI.
- [ ] **PR-2 (Integração):** Camada `api/` conectada aos endpoints reais `BACK-02` e `BACK-06`, mantendo cobertura mínima no CI.
- [ ] Documentação criada em `logbuster-web/docs/features/avaliacoes-e-interacoes.md`.
- [ ] Card `[FRONT-03]` atualizado com os critérios de ordenação/interações e link para esta SPEC.
