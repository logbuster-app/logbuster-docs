# SPEC-BACK-00 — Setup do Repositório e Arquitetura Base (`logbuster-api`)

- **Status:** Approved
- **Card(s) Trello:** `[BACK-00]` *(Card de Setup do Repositório Back-end)*
- **Repositório:** `logbuster-api`
- **Responsável:** Marcelo Palmeira
- **Dependências:** [`SPEC-ARCH-01`](../transversais/SPEC-ARCH-01-arquitetura-ci-quality-gate.md), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md)

---

## 1. Contexto e Objetivo
Inicializar o repositório `logbuster-api` em **FastAPI (Python)** com arquitetura modular em camadas, persistência relacional pronta para uso local imediato, CORS configurado para o Next.js, handlers globais de erro padronizados, pipeline de CI com cobertura mínima de 80% e documentação em Docsify — destravando o desenvolvimento paralelo entre os domínios `movies` (Marcelo) e `reviews` (João Victor).

## 2. Escopo
- **Inclui:** Estrutura de pastas por módulos (`movies`, `reviews`, `core`), configuração via variáveis de ambiente (`.env.example`), banco relacional via SQLAlchemy 2.0 (SQLite local por padrão em `DATABASE_URL`, compatível com PostgreSQL), cliente HTTP assíncrono/síncrono (`httpx`) isolado para OMDb, rota `GET /health`, GitHub Actions (`ci.yml`) e Docsify em `docs/`.
- **Não inclui:** Sistema de usuários/JWT, mensageria assíncrona ou workers externos.

## 3. Regras de Arquitetura
- **RN-01 (Isolamento de Domínios para Paralelismo sem Conflito):**
  ```text
  app/
    main.py                 # App FastAPI, CORS, handlers globais de erro, registro de routers
    core/
      config.py             # Settings via variáveis de ambiente (OMDB_API_KEY, DATABASE_URL, CORS_ORIGINS)
      database.py           # Engine, SessionLocal e Base declarativa SQLAlchemy
      errors.py             # AppError e exception handlers no formato SPEC-API-00
    modules/
      movies/               # Domínio de filmes e integração OMDb (Marcelo)
        router.py
        schemas.py
        service.py
        omdb_client.py
      reviews/              # Domínio de avaliações e interações (João Victor)
        router.py
        models.py
        schemas.py
        repository.py
        service.py
  tests/
    conftest.py             # Banco SQLite em memória isolado por teste + TestClient
  docs/                     # Docsify do repositório back-end
  ```
- **RN-02 (Segurança de Credenciais):** `.gitignore` bloqueia `.env`, `*.db`, `__pycache__`, `.venv` e `.coverage`. Apenas `.env.example` é versionado.
- **RN-03 (Envelope Global de Erros):** `AppError`, `HTTPException` e `RequestValidationError` respondem no formato `{ "error": { "code", "message", "details" } }` definido em [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md).

## 4. Contrato Técnico

### 4.1 Variáveis de Ambiente (`.env.example`)
```ini
DATABASE_URL=sqlite:///./logbuster.db
OMDB_API_KEY=coloque_sua_chave_aqui
OMDB_BASE_URL=https://www.omdbapi.com
CORS_ORIGINS=http://localhost:3000
```

### 4.2 Healthcheck
- **Rota:** `GET /health`
- **Resposta `200 OK`:** `{ "status": "ok" }`

### 4.3 Pipeline de CI (`.github/workflows/ci.yml`)
1. Lint (`ruff check .`).
2. Testes com cobertura mínima (`pytest --cov=app --cov-fail-under=80`).

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Aplicação sobe localmente e responde healthcheck
- **Dado** um ambiente Python configurado a partir do `README.md` e `.env.example`
- **Quando** o servidor FastAPI inicia e recebe `GET /health`
- **Então** retorna HTTP `200` com `{ "status": "ok" }`.

### Cenário 2: CI executa lint e cobertura mínima
- **Dado** um Pull Request aberto em `logbuster-api`
- **Quando** o workflow de CI roda
- **Então** valida lint e exige cobertura `>= 80%` na suíte `pytest`.

## 6. Definição de Pronto (DoD)
- [ ] Estrutura base, banco, CORS, erro padrão e `GET /health` na `main`.
- [ ] Workflow CI funcionando com `pytest --cov-fail-under=80`.
- [ ] Docsify inicial configurado em `logbuster-api/docs/` com guia de setup e arquitetura.
- [ ] *(Pós-PR)* Registrar `ADR-002` em `logbuster-docs/docs/adr/` documentando banco/ORM e arquitetura escolhidos.
