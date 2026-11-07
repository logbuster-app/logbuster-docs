# SPEC-ARCH-01 — Arquitetura dos Repositórios, CI e Quality Gate

- **Status:** Approved
- **Card(s) Trello:** [`LB-003`](https://trello.com/c/933QWLd6), [`LB-006`](https://trello.com/c/8X7ROqsj)
- **Repositório:** Transversal (`logbuster-web`, `logbuster-api`, `logbuster-docs`)
- **Responsável:** João Raphael (Gestão), Marcos (Setup Web), Marcelo (Setup API)
- **Dependências:** [`LB-001`](../../sdd/escopo-parcial-e-radar.md)

---

## 1. Contexto e Objetivo
Padronizar a arquitetura mínima, as regras de isolamento de código e o pipeline automatizado de integração contínua (CI) e cobertura mínima de testes para garantir que nenhum PR quebre a `main` antes da entrega parcial de **10/11/2026**.

## 2. Escopo
- **Inclui:** Estrutura de diretórios do Front e do Back, política de branches/PRs, pipeline GitHub Actions e metas objetivas de cobertura de código.
- **Não inclui:** Deploy produtivo em nuvem paga ou configuração obrigatória de servidores externos pesados de análise estática nesta entrega parcial.

## 3. Regras de Negócio e Arquitetura
- **RN-01 (Isolamento por Domínio no Front):** `logbuster-web` organiza código em `src/features/<domínio>` (`movies`, `reviews`) expondo apenas `index.ts`; rotas em `src/app` são finas e `src/shared` jamais importa de `features`.
- **RN-02 (Arquitetura em Camadas por Módulo no Back):** `logbuster-api` separa responsabilidades em **Routers/Controllers HTTP** (rotas e status HTTP), **Schemas** (Pydantic Request/Response), **Services** (regras de negócio, cálculo de nota média, validações de voto) e **Repositories / Clients Externos** (banco de dados e cliente HTTP da OMDb), agrupados por domínio (`movies`, `reviews`) e `shared`/`core` (config, banco, erros).
- **RN-03 (Mappers de Domínio):** Nem o Front nem os consumidores do Back dependem do formato cru da OMDb (`Title`, `Poster: "N/A"`). A normalização ocorre sempre na fronteira de integração.
- **RN-04 (Segurança de Segredos):** Nenhum arquivo `.env` real ou chave de API pode ser versionado; todo repositório mantém `.env.example` documentado.
- **RN-05 (Quality Gate Obrigatório em PR):** Todo merge na `main` exige CI verde e revisão por ao menos 1 outro membro da equipe.

## 4. Contrato Técnico de CI e Cobertura Mínima (`LB-006`)

| Critério | `logbuster-web` (Implementado) | `logbuster-api` (Obrigatório em `BACK-00`) |
| :--- | :--- | :--- |
| **Gatilhos CI** | `pull_request` e `push` em `main` | `pull_request` e `push` em `main` |
| **Lint & Tipos** | `npm run lint` (ESLint 9) + `npm run typecheck` (`tsc --noEmit`) | `ruff check .` (ou `flake8`/`black`) |
| **Testes & Cobertura** | `vitest run --coverage` (**80%** lines/functions/statements, **70%** branches) | `pytest --cov=app --cov-fail-under=80` (**80%** de cobertura mínima) |
| **Build / Boot Check** | `npm run build` | Import/boot da aplicação FastAPI + teste de `GET /health` |
| **Documentação** | `docs/` em Docsify | `docs/` em Docsify |

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: PR com cobertura abaixo do mínimo ou teste quebrando
- **Dado** um Pull Request aberto em `logbuster-web` ou `logbuster-api`
- **Quando** qualquer teste falhar ou a cobertura cair abaixo do limiar configurado (`80%`)
- **Então** o job do GitHub Actions falha com código diferente de zero e bloqueia a aprovação de merge.

### Cenário 2: Execução local reprodutível sem segredos no Git
- **Dado** um clone limpo do repositório
- **Quando** o desenvolvedor copia `.env.example`, instala as dependências e executa a suíte de testes automatizados (com mocks de rede)
- **Então** todos os testes executam e passam sem depender de chave real da OMDb em ambiente de CI.

## 6. Definição de Pronto (DoD)
- [x] Pipeline e arquitetura de `logbuster-web` validados na `main`.
- [ ] Pipeline e arquitetura de `logbuster-api` entregues em [`SPEC-BACK-00`](../back/SPEC-BACK-00-setup-arquitetura-api.md).
- [ ] Cards `LB-003` e `LB-006` com link para esta SPEC.
