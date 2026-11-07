# SPEC-FRONT-04 — Modal de Cadastro de Avaliações

- **Status:** Approved
- **Card(s) Trello:** [`[FRONT-04]`](https://trello.com/c/HYnwndSj)
- **Repositório:** `logbuster-web`
- **Responsável:** Thyago Viana
- **Dependências:** [`SPEC-BACK-01`](../back/SPEC-BACK-01-post-cadastro-avaliacao.md), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md)

---

## 1. Contexto e Objetivo
Permitir que o usuário cadastre uma avaliação estruturada diretamente na tela do filme (`/movies/[id]`) através de um modal acessível, informando sua nota em estrelas (`0,5` a `5,0`), até 2 pontos de destaque da obra, seu nome/apelido, o texto da análise e um checkbox provisório indicando se o texto contém spoiler.

> **Estratégia de Desacoplamento do Back-end (Entrega em 2 PRs — conforme subtarefas do card):**
> Para que o desenvolvimento do modal avance hoje sem esperar a conclusão de `BACK-01`:
> - **PR-1 (Mock-First — Visual, Validações e Mock de Contrato):** Implementar `ReviewModal`, `StarRatingInput` (`0,5–5,0`), `HighlightPicker` (6 opções, máx. 2) e validações consumindo um adapter mockado em `src/features/reviews/api/reviews.mock.ts` (controlado por `NEXT_PUBLIC_USE_API_MOCKS=true`), simulando latência, retorno `201` no formato exato da API e inserção local na lista da tela.
> - **PR-2 (Integração Real):** Conectar `createReview` ao endpoint real `POST /api/v1/reviews` (`BACK-01`) assim que o card de back-end estiver disponível, sem alterar os componentes visuais.

## 2. Escopo
- **Inclui:** Botão **"Avaliar filme"** na ficha do filme, componente de modal acessível (`ReviewModal`), seletor interativo de estrelas de `0,5` a `5,0`, seletor de até 2 destaques entre as 6 opções fechadas, input de nome do autor, textarea de descrição, checkbox provisório de spoiler, integração com `POST /api/v1/reviews` (`BACK-01`) e callback para atualizar a lista/média da tela após sucesso.
- **Não inclui:** Classificação Análise Séria vs. Reação-Piada, detecção automática de spoiler por IA, marcação de spoiler inline em trechos do texto ou autenticação de conta.

## 3. Regras de Negócio
- **RN-01 (Escala Oficial de Estrelas):** Obrigatório selecionar entre **`0,5` e `5,0`** em incrementos de **`0,5`** (`0.5, 1.0, 1.5, ..., 5.0`). Nota `0` (sem estrelas) não pode ser submetida.
- **RN-02 (Lista Completa de Pontos de Destaque — Máximo 2):** O usuário pode selecionar **0, 1 ou no máximo 2** aspectos entre as **6 opções oficiais**:
  1. **Trilha Sonora** (`soundtrack`)
  2. **Fotografia** (`cinematography`)
  3. **Roteiro** (`screenplay`)
  4. **Atuação** (`acting`)
  5. **Direção** (`direction`)
  6. **Efeitos Especiais** (`visual_effects`)
  - Ao atingir 2 chips selecionados, os demais ficam desabilitados até que o usuário desmarque um dos escolhidos.
- **RN-03 (Campos de Texto):**
  - **Nome / Apelido (`authorName`):** Obrigatório, `2` a `50` caracteres (após `trim()`). Sugestão UX: persistir o último nome usado em `localStorage` para facilitar novas avaliações na demo.
  - **Descrição da avaliação (`content`):** Obrigatória, mínimo `10` e máximo `2000` caracteres (após `trim()`), exibindo contador ou dica clara.
- **RN-04 (Spoiler Provisório):** Checkbox booleano opcional *"Esta avaliação contém spoilers"* (`default: false`).
- **RN-05 (Preservação de Dados em Falha):** Se a chamada `POST` falhar (`422`, `500` ou erro de rede), o modal permanece aberto exibindo o erro (`role="alert"`) **sem apagar** o texto e as seleções digitadas pelo usuário. Em caso de sucesso (`201`), fecha o modal, limpa o formulário e dispara atualização da lista de avaliações e nota média.
- **RN-06 (Acessibilidade do Modal):** Usa semântica `role="dialog"` com `aria-modal="true"`, fecha com tecla `Escape` ou botão Cancelar (exceto enquanto `submitting === true`).

## 4. Contrato Técnico (`src/features/reviews`)

### 4.1 Payload Enviado para `POST /api/v1/reviews` (`BACK-01`)
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

### 4.2 Arquivos Esperados
- `src/features/reviews/api/createReview.ts` (ou `reviewsApi.ts`): função `createReview(input)` mapeando o retorno `201` para `Review`.
- `src/features/reviews/components/StarRatingInput.tsx`: seletor acessível de `0,5` a `5,0` usando token `text-secondary`.
- `src/features/reviews/components/HighlightPicker.tsx`: chips das 6 opções limitando até 2 escolhas.
- `src/features/reviews/components/ReviewModal.tsx`: formulário completo, validações client-side e estados de submissão.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Cadastro válido de avaliação com meia estrela e 2 destaques
- **Dado** que o modal de avaliação está aberto para o filme `tt0133093`
- **Quando** o usuário informa nome `"Thyago"`, seleciona `4,5` estrelas, marca `"Direção"` e `"Efeitos Especiais"`, marca `"Contém spoilers"`, escreve uma análise válida e clica em **"Publicar avaliação"**
- **Então** o Front envia `POST /api/v1/reviews`, fecha o modal ao receber `201 Created` e notifica a tela do filme para exibir a nova avaliação.

### Cenário 2: Bloqueio de 3º ponto de destaque
- **Dado** que o usuário já marcou `"Trilha Sonora"` e `"Fotografia"` no modal
- **Quando** tenta selecionar `"Roteiro"`
- **Então** a interface impede a terceira seleção (mantendo apenas 2 selecionados) até que um dos dois atuais seja desmarcado.

### Cenário 3: Erro de validação ou falha do servidor não apaga texto
- **Dado** que o usuário preencheu todo o formulário e clicou em **"Publicar avaliação"**
- **Quando** o backend responde erro de rede ou `500`
- **Então** uma mensagem de erro é exibida dentro do modal e o texto escrito pelo usuário permanece intacto para nova tentativa.

## 6. Definição de Pronto (DoD)
- [ ] **PR-1 (Mock-First):** `ReviewModal`, seletor `0,5–5,0` e chips dos 6 destaques funcionando sobre `reviews.mock.ts` (seguindo [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md)) com testes passando no CI.
- [ ] **PR-2 (Integração):** `createReview` conectado ao `POST /api/v1/reviews` (`BACK-01`) preservando validações e cobertura mínima no CI.
- [ ] Documentação criada em `logbuster-web/docs/features/cadastro-de-avaliacao.md`.
- [ ] Card `[FRONT-04]` atualizado com a lista completa dos 6 destaques e link para esta SPEC.
