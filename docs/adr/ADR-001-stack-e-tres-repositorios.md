# ADR-001 — Stack Tecnológica e Divisão em Três Repositórios

- **Data:** 2026-11-07
- **Status:** Aceita
- **Autores / Decisores:** João Raphael e Equipe Alpha
- **PR(s) / Card(s) relacionados:** [`LB-003`](https://trello.com/c/933QWLd6), PR #1 de `logbuster-web`

---

## Contexto
O time precisava fechar rapidamente a stack de Front-end, Back-end e Documentação para executar a entrega parcial de 10/11 em paralelo entre 3 desenvolvedores de Front e 2 de Back, mantendo especificações SDD e ADRs acessíveis a toda a equipe sem misturar governança transversal dentro apenas do repositório de interface web.

## Decisão
1. **Três repositórios sob a organização `logbuster-app`:**
   - **`logbuster-web`**: Front-end em **Next.js (App Router) + TypeScript + Tailwind CSS v4**, organizado por **features** (`src/features/*`, `src/shared/*`), testado com **Vitest + Testing Library**.
   - **`logbuster-api`**: Back-end em **FastAPI (Python)**, validado com Pydantic e testado com **pytest**.
   - **`logbuster-docs`**: Repositório central para **SPECs SDD**, contratos HTTP transversais, Identidade Visual e **ADRs**.
2. ** Ferramenta de documentação padronizada:** **Docsify** nos três repositórios (zero etapa de build estático complexo, leitura direta de arquivos Markdown).

## Alternativas Consideradas
1. **Angular no Front-end:** Cogitado em conversa inicial, mas descartado em favor do Next.js pela familiaridade do time, suporte nativo a App Router/SSR e ecossistema moderno de testes com Vitest.
2. **Manter SPECs e ADRs dentro de `logbuster-web/docs`:** Descartado para artefatos transversais e de Back-end, pois forçaria desenvolvedores de API a abrirem PRs no repositório Next.js para documentar endpoints e decisões de servidor.

## Consequências e Impactos
- **Positivas:** Front e Back evoluem com pipelines de CI independentes, enquanto contratos e ADRs permanecem centralizados em `logbuster-docs`.
- **Trade-offs:** Ao concluir tasks que alteram o fluxo de integração (como migrar consultas da OMDb do Front para o Back), o PR no repositório de código deve atualizar sua documentação local e manter conformidade com a SPEC em `logbuster-docs`.
