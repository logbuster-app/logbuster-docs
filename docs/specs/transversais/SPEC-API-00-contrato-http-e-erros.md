# SPEC-API-00 — Contrato HTTP Geral, Padrão de Erros e Identificação de Visitante

- **Status:** Approved
- **Card(s) Trello:** [`LB-004`](https://trello.com/c/6j1DVkVQ), [`LB-002`](https://trello.com/c/yeLLPaIy) *(recorte da entrega parcial)*
- **Repositório:** Transversal (`logbuster-api` ↔ `logbuster-web`)
- **Responsável:** João Raphael, Marcelo, João Victor
- **Dependências:** [`SPEC-ARCH-01`](SPEC-ARCH-01-arquitetura-ci-quality-gate.md)

---

## 1. Contexto e Objetivo
Estabelecer um contrato HTTP único entre o Front-end (Next.js) e o Back-end (FastAPI) antes da implementação paralela dos endpoints `BACK-01` a `BACK-07` e das telas `FRONT-02` a `FRONT-06`, evitando retrabalho por divergência de nomes de campos, formato de erro, CORS ou regras de voto/spoiler sem login.

## 2. Escopo
- **Inclui:** Prefixo versionado, convenção JSON, CORS, envelope único de erros, identificação anônima de visitante (`X-Visitor-Id`) e enums compartilhados de avaliação/interação.
- **Não inclui:** Autenticação JWT/sessão de conta de usuário, controle de visibilidade de perfil (`público/amigos/privado`) ou moderação (fora de escopo de 10/11).

## 3. Regras de Negócio Transversais (`LB-002` / `LB-004`)
- **RN-01 (Identificador Canônico de Filme):** Todo filme é referenciado pelo `imdbID` da OMDb (formato `^tt\d{7,10}$`, ex.: `"tt0133093"`), tanto nas rotas do Back quanto nas rotas do Front (`/movies/[id]`).
- **RN-02 (Versionamento e Formato JSON):** Todas as rotas de negócio usam o prefixo `/api/v1` com `Content-Type: application/json` e chaves em `snake_case`. No `logbuster-web`, a camada `api/` de cada feature converte `snake_case` para tipos de domínio em `camelCase`.
- **RN-03 (Visitante sem Login na Parcial):** Como contas de usuário estão fora do escopo de 10/11:
  1. No cadastro de avaliação (`BACK-01`), o autor informa `author_name` (`2` a `50` caracteres).
  2. Nas interações (`BACK-06`) e leituras de avaliações (`BACK-02`, `BACK-03`), o navegador gera uma única vez um UUID v4 salvo em `localStorage` (`logbuster:visitor_id`) e o envia no header HTTP **`X-Visitor-Id`**. Isso permite *toggle* de votos (1 voto de cada tipo por visitante por avaliação) e destacar na UI quais botões o visitante já marcou.
- **RN-04 (Enums Compartilhados do Domínio de Avaliações):**
  - **Escala de estrelas (`rating`):** `0.5` a `5.0`, exclusivamente em múltiplos de `0.5` (`0.5, 1.0, 1.5, 2.0, 2.5, 3.0, 3.5, 4.0, 4.5, 5.0`).
  - **Pontos de destaque (`highlights`):** Lista com **no mínimo `0` e no máximo `2` valores únicos** pertencentes ao conjunto:
    - `"soundtrack"` (*Trilha Sonora*)
    - `"cinematography"` (*Fotografia*)
    - `"screenplay"` (*Roteiro*)
    - `"acting"` (*Atuação*)
    - `"direction"` (*Direção*)
    - `"visual_effects"` (*Efeitos Especiais*)
  - **Tipos de interação (`InteractionType`):** Exclusivamente `"like"`, `"spoiler"` e `"well_argued"`. *(Não existe `dislike`.)*
- **RN-05 (Regra Provisória de Spoiler na Parcial):** Uma avaliação deve ser exibida inicialmente oculta no Front-end quando `is_spoiler == true` (marcado pelo autor no modal) **ou** quando `counts.spoiler >= 3` (sinalização da comunidade). O leitor sempre pode clicar localmente para revelar o texto sem alterar o estado no servidor.

---

## 4. Contrato Técnico

### 4.1 Mapa Consolidado de Rotas do `logbuster-api`

| Card | Método e Rota | Finalidade |
| :--- | :--- | :--- |
| `BACK-00` | `GET /health` | Healthcheck da API (`{"status": "ok"}`) |
| `BACK-01` | `POST /api/v1/reviews` | Cadastrar nova avaliação de um filme |
| `BACK-02` | `GET /api/v1/movies/{movie_id}/reviews` | Listar e ordenar avaliações de um filme |
| `BACK-03` | `GET /api/v1/reviews/{review_id}` | Obter detalhes de uma avaliação específica |
| `BACK-04` | `GET /api/v1/movies` | Buscar filmes integrando à OMDb pelo backend |
| `BACK-05` | `GET /api/v1/movies/{movie_id}/rating` *(+ campos em `BACK-04`/`07`)* | Obter `rating_avg` e `rating_count` do filme |
| `BACK-06` | `POST /api/v1/reviews/{review_id}/interactions` | Alternar (*toggle*) `like`, `spoiler` ou `well_argued` |
| `BACK-07` | `GET /api/v1/movies/{movie_id}` | Obter detalhes completos de um filme (OMDb + nota média LogBuster) |

### 4.2 CORS
O `logbuster-api` deve habilitar `CORSMiddleware` permitindo origens configuradas via `CORS_ORIGINS` (padrão local `http://localhost:3000`), métodos `GET, POST, OPTIONS` e headers `Content-Type, X-Visitor-Id`.

### 4.3 Envelope Único de Erro (`4xx` e `5xx`)

Toda resposta de erro do `logbuster-api` (incluindo sobrescrita de `RequestValidationError` do FastAPI) deve retornar:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "A nota deve estar entre 0.5 e 5.0 em passos de 0.5.",
    "details": [
      {
        "field": "rating",
        "issue": "Valor inválido: 0.3"
      }
    ]
  }
}
```

| HTTP Status | `error.code` | Quando usar |
| :--- | :--- | :--- |
| `400 Bad Request` | `BAD_REQUEST` | Header obrigatório ausente (ex.: sem `X-Visitor-Id` em `BACK-06`) ou query inválida |
| `404 Not Found` | `NOT_FOUND` | `review_id` inexistente no banco ou `movie_id` não encontrado na OMDb |
| `422 Unprocessable Entity` | `VALIDATION_ERROR` | Payload viola regras de domínio (estrelas fora de `0.5..5.0`, >2 destaques, etc.) |
| `502 Bad Gateway` | `UPSTREAM_ERROR` | Falha de rede, timeout ou chave inválida ao consultar a OMDb pelo backend |
| `500 Internal Server Error` | `INTERNAL_ERROR` | Erro inesperado interno (nunca vazar stacktrace cru para o cliente) |

---

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Erro de validação segue o envelope padronizado
- **Dado** que o cliente envia um payload inválido para qualquer rota `/api/v1/*`
- **Quando** o FastAPI rejeita a requisição com status `422`
- **Então** o corpo JSON contém o objeto raiz `error` com `code: "VALIDATION_ERROR"`, `message` em português claro e lista `details`.

### Cenário 2: Chamada cross-origin do Front local funciona com `X-Visitor-Id`
- **Dado** o `logbuster-web` rodando em `http://localhost:3000` e `logbuster-api` em `http://localhost:8000`
- **Quando** o navegador dispara `OPTIONS` (preflight) seguido de `POST` enviando o header `X-Visitor-Id`
- **Então** a API responde com os headers CORS apropriados sem bloqueio no navegador.

## 6. Definição de Pronto (DoD)
- [ ] Handlers globais de exceção e CORS implementados no `BACK-00`.
- [ ] Utilitário HTTP e gerador de `X-Visitor-Id` alinhados no `logbuster-web`.
- [ ] Card `LB-004` vinculado a esta SPEC.
