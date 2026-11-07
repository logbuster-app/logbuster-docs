# SPEC-BACK-01 — Endpoint 1: `POST` Cadastro de Avaliações

- **Status:** Approved
- **Card(s) Trello:** [`[BACK-01]`](https://trello.com/c/tgZpLpZ7)
- **Repositório:** `logbuster-api`
- **Responsável:** João Victor
- **Dependências:** [`SPEC-BACK-00`](SPEC-BACK-00-setup-arquitetura-api.md), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md)

---

## 1. Contexto e Objetivo
Expor o endpoint de criação de uma nova avaliação de filme no LogBuster, persistindo a nota em estrelas (`0,5` a `5,0`), descrição textual, nome do autor, flag provisória de spoiler e até 2 pontos de destaque da obra.

## 2. Escopo
- **Inclui:** Modelo relacional `Review` (e armazenamento dos `highlights`), validações de domínio via Pydantic v2, rota `POST /api/v1/reviews` retornando `201 Created`, testes unitários/integração e documentação no Docsify do `logbuster-api`.
- **Não inclui:** Análise de spoiler por IA, classificação Análise Séria/Reação-Piada ou autenticação de usuário.

## 3. Regras de Negócio
- **RN-01 (`movie_id`):** Deve seguir o padrão de ID IMDb (`^tt\d{7,10}$`, ex.: `"tt0133093"`).
- **RN-02 (`rating`):** Obrigatório entre `0.5` e `5.0`, exclusivamente em múltiplos de `0.5` (`(rating * 2) % 1 == 0`). Valores como `0`, `0.3`, `5.5` ou negativos retornam `422 VALIDATION_ERROR`.
- **RN-03 (`highlights`):** Lista opcional contendo **de `0` a `2` valores únicos** pertencentes ao enum:
  `soundtrack`, `cinematography`, `screenplay`, `acting`, `direction`, `visual_effects`.
  Enviar 3 ou mais destaques, itens repetidos ou valor fora do enum retorna `422 VALIDATION_ERROR`.
- **RN-04 (`author_name` e `content`):** Após `strip()`, `author_name` deve ter entre `2` e `50` caracteres e `content` entre `10` e `2000` caracteres.
- **RN-05 (Estado Inicial):** Toda avaliação recém-criada nasce com `id` (UUID v4), `created_at` em UTC ISO-8601 e contadores de interação zerados (`like: 0, spoiler: 0, well_argued: 0`).

## 4. Contrato Técnico

- **Método e Rota:** `POST /api/v1/reviews`
- **Request Body (`application/json`):**
  ```json
  {
    "movie_id": "tt0133093",
    "author_name": "Thyago",
    "rating": 4.5,
    "content": "Direção precisa e efeitos visuais que revolucionaram o gênero.",
    "is_spoiler": false,
    "highlights": ["direction", "visual_effects"]
  }
  ```
- **Response `201 Created`:**
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
      "like": 0,
      "spoiler": 0,
      "well_argued": 0
    },
    "viewer_interactions": [],
    "created_at": "2026-11-08T18:30:00Z"
  }
  ```
- **Erros:** `422 Unprocessable Entity` no envelope padrão (`SPEC-API-00`).

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Criação de avaliação válida
- **Dado** um payload válido com `rating: 4.5` e `highlights: ["direction", "visual_effects"]`
- **Quando** `POST /api/v1/reviews` é chamado
- **Então** persiste no banco e retorna HTTP `201` com `id` gerado e contadores zerados.

### Cenário 2: Rejeição de nota fora do passo de 0,5 ou excesso de destaques
- **Dado** um payload com `rating: 0` (ou `4.3`) ou com 3 itens em `highlights`
- **Quando** `POST /api/v1/reviews` é chamado
- **Então** retorna HTTP `422` com `error.code == "VALIDATION_ERROR"` e nada é salvo no banco.

## 6. Definição de Pronto (DoD)
- [ ] Endpoint `POST /api/v1/reviews` implementado em `app/modules/reviews`.
- [ ] Testes cobrindo criação válida e todos os limites de validação (`rating`, `highlights`, `movie_id`, `content`) passando no CI (`>= 80%`).
- [ ] Endpoint documentado no Docsify de `logbuster-api/docs/`.
- [ ] Card `[BACK-01]` atualizado com descrição e link para esta SPEC.
