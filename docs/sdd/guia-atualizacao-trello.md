# Guia Prático de Atualização e Remanejamento do Quadro Trello

Este guia consolida exatamente o que ajustar nos **cards abertos** do Trello para eliminar todas as inconsistências mapeadas e vincular cada card à sua respectiva SPEC em `logbuster-docs`.

> **Padrão de link para colar no campo *"Spec de referência documentada?"* após o push na `main`:**
> `Sim — https://github.com/logbuster-app/logbuster-docs/blob/main/docs/specs/<pasta>/<arquivo>.md`

---

## 1. Cards Novos a Criar (Documentação de Setup)

| Card sugerido | Lista | Membro | SPEC |
| :--- | :--- | :--- | :--- |
| **`[FRONT-00] Setup Next.js, arquitetura por features, CI e Docsify`** | *Concluído* | Marcos Vinicius | `SPEC-FRONT-00` |
| **`[BACK-00] Setup FastAPI, arquitetura base, banco, CI e Docsify`** | *Pronto para fazer* | Marcelo Palmeira | `SPEC-BACK-00` |

---

## 2. Ajustes nos Cards Operacionais Abertos (`FRONT` e `BACK`)

1. **`[FRONT-01]` (#65 — Marcos):** Já em *Concluído*. Apenas preencher `Spec de referência documentada?: Sim (SPEC-FRONT-01)` e concluir revisão com Elton e Thyago.
2. **`[FRONT-02]` (#66 — Elton):** Preencher `Spec Documentada: Sim (SPEC-FRONT-02)`.
3. **`[FRONT-03]` (#67 — Elton):** Atualizar os **Critérios de aceite** incluindo explicitamente:
   - Seletor de ordenação das avaliações (`Mais bem argumentadas` como padrão, `Mais curtidas`, `Mais recentes`, `Maior/Menor nota`);
   - Botões de interação **`BEM ARGUMENTADA`**, **`LIKE`** e **`SPOILER`** (consumindo `[BACK-06]`);
   - Ocultação inicial de análises marcadas como spoiler com botão local para revelar;
   - Link para `SPEC-FRONT-03`.
4. **`[FRONT-04]` (#68 — Thyago):** Substituir a lista cortada de pontos de destaque pela lista oficial completa (**até 2** entre: *Trilha Sonora, Fotografia, Roteiro, Atuação, Direção e Efeitos Especiais*), manter estrelas `0,5 a 5,0` e vincular `SPEC-FRONT-04`.
5. **`[FRONT-05]` (#75 — atribuir a Marcos):** Atribuir ao **Marcos**, exigir atualização do Docsify do Front ao migrar a busca para o Back e vincular `SPEC-FRONT-05`.
6. **`[FRONT-06]` (#76 — atribuir a Marcos):** Atribuir ao **Marcos**, preencher descrição consumindo `[BACK-07]` (metadados + nota média do Back) + atualização do Docsify e vincular `SPEC-FRONT-06`.
7. **`[BACK-01]` (#69 — João Victor):** Preencher descrição com `POST /api/v1/reviews` e vincular `SPEC-BACK-01`.
8. **`[BACK-02]` (#70 — João Victor):** Preencher descrição com `GET /api/v1/movies/{movie_id}/reviews` (incluindo parâmetro `sort` com `well_argued` padrão) e vincular `SPEC-BACK-02`.
9. **`[BACK-03]` (#71 — João Victor):** Preencher descrição com `GET /api/v1/reviews/{review_id}` e vincular `SPEC-BACK-03`.
10. **`[BACK-04]` (#72 — Marcelo):** Preencher descrição com `GET /api/v1/movies?q=&page=` e vincular `SPEC-BACK-04`.
11. **`[BACK-05]` (#73 — Marcelo):** Preencher descrição com cálculo de `rating_avg`/`rating_count` em `GET /api/v1/movies` + `GET /api/v1/movies/{movie_id}/rating` e vincular `SPEC-BACK-05`.
12. **`[BACK-06]` (#74 — atribuir a João Victor):**
    - **Corrigir título** de:
      `[BACK-06] Endpoint 6 -- POST cadastro de interação (LIKE, DISLIKE, SPOILER) com comentários de avaliações`
      para:
      `[BACK-06] Endpoint 6 — POST cadastro/toggle de interação (LIKE, SPOILER, BEM ARGUMENTADA) em avaliações`
    - Atribuir a **João Victor** e vincular `SPEC-BACK-06`.
13. **`[BACK-07]` (#77 — atribuir a Marcelo):** Atribuir a **Marcelo**, preencher descrição com `GET /api/v1/movies/{movie_id}` (OMDb + `rating_avg`) e vincular `SPEC-BACK-07`.

---

## 3. Ajustes nos Cards `LB-001` a `LB-008`

- Atribuir **João Raphael** como responsável de gestão e ajustar as datas vencidas (`03/11`–`06/11`) para **`08/11/2026`** (mantendo `LB-002` com escopo parcial resolvido em `SPEC-API-00` e regras completas para `11/11+`).
- No **`LB-001`**, preencher validador do recorte do MVP: **João Raphael + Equipe Alpha** e vincular `docs/sdd/escopo-parcial-e-radar.md`.
- Vincular `LB-004` → `SPEC-API-00`, `LB-005`/`LB-007` → `fluxo-sdd-e-dod.md`, `LB-006` → `SPEC-ARCH-01` e `LB-008` → `SPEC-UI-01`.
