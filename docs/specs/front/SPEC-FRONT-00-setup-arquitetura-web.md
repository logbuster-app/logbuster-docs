# SPEC-FRONT-00 — Setup do Projeto e Arquitetura Web (`logbuster-web`)

- **Status:** Implemented (As-Built — PR #1)
- **Card(s) Trello:** `[FRONT-00]` *(Card de Setup/Inicialização Front para documentação)*
- **Repositório:** `logbuster-web`
- **Responsável:** Marcos Vinicius
- **Dependências:** [`SPEC-ARCH-01`](../transversais/SPEC-ARCH-01-arquitetura-ci-quality-gate.md), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md)

---

## 1. Contexto e Objetivo
Inicializar a aplicação web do LogBuster em Next.js (App Router) com arquitetura modular por features, tokens visuais semânticos, cliente de ambiente seguro, documentação em Docsify e pipeline de CI com cobertura mínima automatizada.

## 2. Escopo
- **Inclui:** Estrutura `src/app`, `src/features/{movies,reviews}`, `src/shared`, Tailwind CSS v4 com `@theme`, Vitest + Testing Library, ESLint, `.env.example`, GitHub Actions (`ci.yml`) e Docsify em `docs/`.
- **Não inclui:** Autenticação de usuários, gerenciamento de estado global externo desnecessário ou rotas fora do Épico de Filmes.

## 3. Regras de Negócio e Arquitetura
- **RN-01:** Rotas em `src/app` são finas e apenas compõem componentes exportados pelo `index.ts` público de cada feature.
- **RN-02:** `src/shared` nunca importa de `src/features`, e uma feature nunca importa diretamente arquivos internos de outra feature.
- **RN-03:** Variáveis públicas (`NEXT_PUBLIC_OMDB_API_KEY`, `NEXT_PUBLIC_OMDB_BASE_URL`, `NEXT_PUBLIC_API_BASE_URL`) são lidas exclusivamente por nome literal em `src/shared/config/env.ts`.

## 4. Contrato Técnico (Entregue)
- **Scripts obrigatórios (`package.json`):** `dev`, `build`, `lint`, `typecheck`, `test`, `test:coverage`, `check` e `docs`.
- **Componentes base em `src/shared`:** `SiteHeader`, `Spinner`, `StateMessage` e classe `ApiError` (`config | network | upstream`).
- **Thresholds Vitest (`vitest.config.mts`):** `lines: 80`, `functions: 80`, `statements: 80`, `branches: 70`.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Verificação completa local e no CI
- **Dado** o repositório `logbuster-web` instalado com `npm ci`
- **Quando** `npm run check` e `npm run build` são executados
- **Então** lint, checagem de tipos, suíte Vitest com cobertura mínima e build de produção concluem sem erros.

## 6. Definição de Pronto (DoD)
- [x] Código e estrutura por features mergeados na `main` (PR #1).
- [x] Workflow `.github/workflows/ci.yml` ativo.
- [x] Docsify inicial publicado em `logbuster-web/docs/`.
