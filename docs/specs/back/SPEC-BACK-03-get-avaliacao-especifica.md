# SPEC-BACK-03 — Endpoint 3: `GET` Leitura de Avaliação Específica

- **Status:** Approved
- **Card(s) Trello:** [`[BACK-03]`](https://trello.com/c/ld5BxwLg)
- **Repositório:** `logbuster-api`
- **Responsável:** João Victor
- **Dependências:** [`SPEC-BACK-01`](SPEC-BACK-01-post-cadastro-avaliacao.md), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md)

---

## 1. Contexto e Objetivo
Permitir consultar individualmente todos os dados e contadores atualizados de uma avaliação específica a partir do seu identificador único (`review_id`).

## 2. Escopo
- **Inclui:** Rota `GET /api/v1/reviews/{review_id}` retornando o schema completo `ReviewResponse` (incluindo `counts` e `viewer_interactions` se `X-Visitor-Id` for enviado) e erro `404 NOT_FOUND` padronizado caso não exista.
- **Não inclui:** Edição, exclusão ou comentários em avaliações.

## 3. Regras de Negócio
- **RN-01:** Se `review_id` existir no banco, retorna `200 OK` com exatamente a mesma estrutura de item usada em `BACK-01` e `BACK-02`.
- **RN-02:** Se `review_id` não existir, retorna `404 Not Found` usando o envelope de erro de `SPEC-API-00`.

## 4. Contrato Técnico

- **Método e Rota:** `GET /api/v1/reviews/{review_id}`
- **Headers Opcionais:** `X-Visitor-Id: <uuid>`
- **Response `200 OK`:**
  ```json
  {
    "id": "c4f7d9d2-9d4e-4b3a-8f55-1f2e3d4c5b6a",
    "movie_id": "tt0133093",
    "author_name": "Thyago",
    "rating": 4.5,
    "content": "Direção precisa e efeitos visuais que revolucionaram o gênero.",
    "is_spoiler": false,
    "highlights": ["direction", "visual_effects"],
    "counts": {
      "like": 4,
      "spoiler": 0,
      "well_argued": 9
    },
    "viewer_interactions": [],
    "created_at": "2026-11-08T18:30:00Z"
  }
  ```
- **Response `404 Not Found`:**
  ```json
  {
    "error": {
      "code": "NOT_FOUND",
      "message": "Avaliação não encontrada.",
      "details": []
    }
  }
  ```

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Consulta de avaliação existente
- **Dado** uma avaliação previamente cadastrada com ID `X`
- **Quando** `GET /api/v1/reviews/X` é requisitado
- **Então** retorna HTTP `200` com todos os campos, destaques e contadores da avaliação `X`.

### Cenário 2: Avaliação inexistente
- **Dado** um `review_id` não cadastrado
- **Quando** `GET /api/v1/reviews/{review_id}` é requisitado
- **Então** retorna HTTP `404` com `error.code == "NOT_FOUND"`.

## 6. Definição de Pronto (DoD)
- [ ] Rota `GET /api/v1/reviews/{review_id}` implementada.
- [ ] Testes de sucesso (`200`) e não encontrado (`404`) passando no CI.
- [ ] Documentação atualizada em `logbuster-api/docs/`.
- [ ] Card `[BACK-03]` atualizado com descrição e link para esta SPEC.
