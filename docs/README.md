# LogBuster — Especificações SDD & Documentação Central

Rede social de avaliações de filmes e séries, inspirada no Letterboxd, que **ordena análises pela qualidade do argumento** (*voto qualificado: "Bem argumentada"*) e destaca os pontos fortes de cada obra.

Este repositório (`logbuster-docs`) concentra o **Specification-Driven Development (SDD)** transversal aos repositórios `logbuster-web` (Next.js) e `logbuster-api` (FastAPI), além do registro de **ADRs**.

---

## Equipe

| Papel | Integrante | Foco na Entrega Parcial (10/11) |
| :--- | :--- | :--- |
| **Gestor de Projeto** | João Raphael | Escopo, Governança SDD (`LB-001` a `LB-008`), Contratos e Quality Gate |
| **Front-end (A)** | Marcos Vinicius | Setup Web, `FRONT-01`, `FRONT-05`, `FRONT-06` |
| **Front-end (B)** | Elton Gonçalves | `FRONT-02`, `FRONT-03` e revisão cruzada |
| **Front-end (C)** | Thyago Viana | `FRONT-04` e revisão cruzada |
| **Back-end (A)** | Marcelo Palmeira | Setup API (`BACK-00`), domínio `movies` (`BACK-04`, `BACK-05`, `BACK-07`) |
| **Back-end (B)** | João Victor | Domínio `reviews` (`BACK-01`, `BACK-02`, `BACK-03`, `BACK-06`) |

---

## Matriz de Rastreabilidade — Cards Abertos × SPECs

### 1. Governança, Arquitetura e Base Técnica (`LB-001` a `LB-008`)

| Card Trello | Título | Documento SDD de Referência |
| :--- | :--- | :--- |
| [`LB-001`](https://trello.com/c/VHT9kZQV) | Validar recorte do MVP e entregáveis na entrega parcial | [Escopo Parcial (10/11) & Radar](sdd/escopo-parcial-e-radar.md) |
| [`LB-002`](https://trello.com/c/yeLLPaIy) | Fechar parâmetros de voto, spoiler e privacidade *(recorte parcial)* | [SPEC-API-00](specs/transversais/SPEC-API-00-contrato-http-e-erros.md) · [SPEC-BACK-06](specs/back/SPEC-BACK-06-post-interacao-avaliacao.md) |
| [`LB-003`](https://trello.com/c/933QWLd6) | Definir stack e arquitetura mínima dos três repositórios | [SPEC-ARCH-01](specs/transversais/SPEC-ARCH-01-arquitetura-ci-quality-gate.md) |
| [`LB-004`](https://trello.com/c/6j1DVkVQ) | Versionar contrato HTTP e padrão de erros | [SPEC-API-00](specs/transversais/SPEC-API-00-contrato-http-e-erros.md) |
| [`LB-005`](https://trello.com/c/Stmyha6J) | Adotar template de SPEC, ADR e fluxo de revisão | [Fluxo SDD & DoD](sdd/fluxo-sdd-e-dod.md) · [SPEC-TEMPLATE](sdd/SPEC-TEMPLATE.md) · [ADR-TEMPLATE](sdd/ADR-TEMPLATE.md) |
| [`LB-006`](https://trello.com/c/8X7ROqsj) | Configurar pipeline de testes, PR e quality gate | [SPEC-ARCH-01](specs/transversais/SPEC-ARCH-01-arquitetura-ci-quality-gate.md) |
| [`LB-007`](https://trello.com/c/IbH6y1hv) | Definir responsáveis e execução do teste manual de cada épico | [Fluxo SDD & DoD](sdd/fluxo-sdd-e-dod.md) |
| [`LB-008`](https://trello.com/c/m7bgLvRQ) | Produzir wireframes, protótipo e sistema de design | [SPEC-UI-01 · Identidade Visual](specs/transversais/SPEC-UI-01-identidade-visual.md) |

### 2. Épico de Filmes — Front-end (`logbuster-web`)

| Card Trello | Título | Responsável | SPEC Vinculada |
| :--- | :--- | :--- | :--- |
| `[FRONT-00]` *(Setup)* | Inicialização Next.js, arquitetura, CI e Docsify | Marcos | [SPEC-FRONT-00](specs/front/SPEC-FRONT-00-setup-arquitetura-web.md) |
| [`[FRONT-01]`](https://trello.com/c/2WAFMwRD) | Busca de filmes e integração provisória com API OMDb | Marcos | [SPEC-FRONT-01](specs/front/SPEC-FRONT-01-busca-filmes-omdb.md) |
| [`[FRONT-02]`](https://trello.com/c/8PfLP55W) | Tela de visualização de filme específico | Elton | [SPEC-FRONT-02](specs/front/SPEC-FRONT-02-tela-filme-especifico.md) |
| [`[FRONT-03]`](https://trello.com/c/pRQiSMGn) | Visualização, ordenação e interação com avaliações | Elton | [SPEC-FRONT-03](specs/front/SPEC-FRONT-03-lista-ordenacao-interacoes-avaliacoes.md) |
| [`[FRONT-04]`](https://trello.com/c/HYnwndSj) | Modal de cadastro de avaliações | Thyago | [SPEC-FRONT-04](specs/front/SPEC-FRONT-04-modal-cadastro-avaliacao.md) |
| [`[FRONT-05]`](https://trello.com/c/rU1a52FF) | Adaptar busca de filmes para consumir `BACK-04`/`BACK-05` | Marcos | [SPEC-FRONT-05](specs/front/SPEC-FRONT-05-adaptar-busca-para-backend.md) |
| [`[FRONT-06]`](https://trello.com/c/MMFGf9pn) | Adaptar tela de filme específico para consumir `BACK-07` | Marcos | [SPEC-FRONT-06](specs/front/SPEC-FRONT-06-adaptar-detalhe-filme-backend.md) |

### 3. Épico de Filmes — Back-end (`logbuster-api`)

| Card Trello | Título | Responsável | SPEC Vinculada |
| :--- | :--- | :--- | :--- |
| `[BACK-00]` *(Setup)* | Setup FastAPI, arquitetura, banco, erro padrão, CI e Docsify | Marcelo | [SPEC-BACK-00](specs/back/SPEC-BACK-00-setup-arquitetura-api.md) |
| [`[BACK-01]`](https://trello.com/c/tgZpLpZ7) | Endpoint 1 — `POST` cadastro de avaliações | João Victor | [SPEC-BACK-01](specs/back/SPEC-BACK-01-post-cadastro-avaliacao.md) |
| [`[BACK-02]`](https://trello.com/c/QgTm932N) | Endpoint 2 — `GET` leitura e ordenação de avaliações | João Victor | [SPEC-BACK-02](specs/back/SPEC-BACK-02-get-lista-avaliacoes.md) |
| [`[BACK-03]`](https://trello.com/c/ld5BxwLg) | Endpoint 3 — `GET` leitura de avaliação específica | João Victor | [SPEC-BACK-03](specs/back/SPEC-BACK-03-get-avaliacao-especifica.md) |
| [`[BACK-04]`](https://trello.com/c/NfQxCkHy) | Endpoint 4 — `GET` filmes com integração à OMDb | Marcelo | [SPEC-BACK-04](specs/back/SPEC-BACK-04-get-busca-filmes-omdb.md) |
| [`[BACK-05]`](https://trello.com/c/yWVNSm4E) | Endpoint 5 — Nota média do filme (`GET` filmes e por ID) | Marcelo | [SPEC-BACK-05](specs/back/SPEC-BACK-05-nota-media-filmes.md) |
| [`[BACK-06]`](https://trello.com/c/GVUsTSLs) | Endpoint 6 — `POST` cadastro/toggle de interação (`LIKE`, `SPOILER`, `BEM ARGUMENTADA`) | João Victor | [SPEC-BACK-06](specs/back/SPEC-BACK-06-post-interacao-avaliacao.md) |
| [`[BACK-07]`](https://trello.com/c/BKdWBAp8) | Endpoint 7 — `GET` consulta de filme específico na OMDb + média | Marcelo | [SPEC-BACK-07](specs/back/SPEC-BACK-07-get-filme-especifico-omdb.md) |

### 4. Entrega de 25/11 — Back-end (`logbuster-api`)

| Card Trello | Escopo | Responsável | SPEC Vinculada |
| :--- | :--- | :--- | :--- |
| [Back A](https://trello.com/c/161rtpgG/81-back-a) | Autenticação, perfil público, foto e fundo personalizados | Marcelo | [SPEC-BACK-08](specs/back/SPEC-BACK-08-auth-perfil-publico.md) |
| [Back B](https://trello.com/c/adnU5yAW/82-back-b) | Resumo, sugestão de spoiler, arquétipo e retrospectiva condicional | João Victor | [SPEC-BACK-09](specs/back/SPEC-BACK-09-ia-e-arquetipo.md) |
