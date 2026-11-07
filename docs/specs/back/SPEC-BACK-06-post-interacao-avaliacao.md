# SPEC-BACK-06 — Endpoint 6: `POST` Cadastro/Toggle de Interação em Avaliação

- **Status:** Approved
- **Card(s) Trello:** [`[BACK-06]`](https://trello.com/c/GVUsTSLs) *(Título alinhado para `LIKE`, `SPOILER`, `BEM ARGUMENTADA`)*
- **Repositório:** `logbuster-api`
- **Responsável:** João Victor
- **Dependências:** [`SPEC-BACK-01`](SPEC-BACK-01-post-cadastro-avaliacao.md), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md)

---

## 1. Contexto e Objetivo
Permitir que leitores registrem ou desfaçam (*toggle*) reações sobre uma avaliação existente — **`like`** (*Curtir*), **`spoiler`** (*Sinalizar spoiler*) e **`well_argued`** (*Bem argumentada*, voto qualificado que alimenta o ranking de mérito de `BACK-02`).

## 2. Escopo
- **Inclui:** Tabela/entidade `ReviewInteraction` com restrição de unicidade `(review_id, visitor_id, type)`, rota `POST /api/v1/reviews/{review_id}/interactions` exigindo header `X-Visitor-Id` e retorno imediato dos contadores atualizados.
- **Não inclui:** `DISLIKE`, comentários de texto em avaliações ou sistema de contas autenticadas.

## 3. Regras de Negócio
- **RN-01 (Tipos Válidos):** Apenas `"like"`, `"spoiler"` e `"well_argued"` são aceitos. Qualquer outro valor (como `"dislike"`) retorna `422 VALIDATION_ERROR`.
- **RN-02 (Identificação Obrigatória do Visitante):** O header `X-Visitor-Id` (não vazio, até 64 caracteres / UUID) é obrigatório. Ausência retorna `400 BAD_REQUEST`.
- **RN-03 (Semântica de Toggle Independente por Tipo):**
  - Se ainda **não existe** registro para `(review_id, visitor_id, type)` → **insere** o registro (`active: true`) e incrementa o contador daquele tipo.
  - Se **já existe** registro para `(review_id, visitor_id, type)` → **remove** o registro (`active: false`) e decrementa o contador daquele tipo.
  - Tipos diferentes são independentes entre si: o mesmo visitante pode marcar simultaneamente `like` e `well_argued` na mesma avaliação (máximo 1 vez cada tipo).
- **RN-04 (Avaliação Inexistente):** Se `review_id` não existir, retorna `404 NOT_FOUND`.

## 4. Contrato Técnico

- **Método e Rota:** `POST /api/v1/reviews/{review_id}/interactions`
- **Headers Obrigatórios:** `X-Visitor-Id: 550e8400-e29b-41d4-a716-446655440000`
- **Request Body (`application/json`):**
  ```json
  {
    "type": "well_argued"
  }
  ```
- **Response `200 OK`:**
  ```json
  {
    "review_id": "c4f7d9d2-9d4e-4b3a-8f55-1f2e3d4c5b6a",
    "type": "well_argued",
    "active": true,
    "counts": {
      "like": 4,
      "spoiler": 0,
      "well_argued": 10
    },
    "viewer_interactions": ["well_argued"]
  }
  ```
- **Erros:** `400 BAD_REQUEST` (sem `X-Visitor-Id`), `404 NOT_FOUND` (`review_id` inválido), `422 VALIDATION_ERROR` (`type` inválido).

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Primeiro clique adiciona voto e segundo clique remove (toggle)
- **Dado** uma avaliação com `well_argued: 0` e um visitante com `X-Visitor-Id: V1`
- **Quando** `V1` envia `POST .../interactions` com `{"type": "well_argued"}` duas vezes seguidas
- **Então** a primeira resposta traz `active: true` e `counts.well_argued: 1`, e a segunda traz `active: false` e `counts.well_argued: 0`.

### Cenário 2: Rejeição de tipo inválido ou falta de `X-Visitor-Id`
- **Dado** uma requisição com `{"type": "dislike"}` ou sem header `X-Visitor-Id`
- **Quando** processada pela API
- **Então** retorna `422` (para tipo inválido) ou `400` (para header ausente) no envelope padrão de erro.

## 6. Definição de Pronto (DoD)
- [ ] Rota `POST /api/v1/reviews/{review_id}/interactions` implementada com constraint única no banco.
- [ ] Testes cobrindo toggle, múltiplos visitantes, tipos independentes, `400`, `404` e `422` passando no CI.
- [ ] Documentação atualizada em `logbuster-api/docs/`.
- [ ] Card `[BACK-06]` renomeado e vinculado a esta SPEC.
