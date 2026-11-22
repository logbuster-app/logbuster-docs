# SPEC-FRONT-08 — Resumo da Comunidade com IA na Ficha do Filme

- **Status:** Em revisão (implementado no [PR #3](https://github.com/logbuster-app/logbuster-web/pull/3) do `logbuster-web`, aguardando merge após o CR-02 do back)
- **Card(s) Trello:** `[FRONT-08]` *(atualizar com o link do card)*
- **Repositório:** `logbuster-web`
- **Responsável:** Marcos Vinicius, com apoio de João Victor (Back B)
- **Dependências:** [`SPEC-BACK-09`](../back/SPEC-BACK-09-ia-e-arquetipo.md) (resumo da comunidade), [`SPEC-FRONT-06`](SPEC-FRONT-06-adaptar-detalhe-filme-backend.md) (ficha do filme), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md), [`CR-02`](../cr/CR-02-spoiler-ia-pos-publicacao.md)
- **Entrega:** congelamento em 24/11/2026

> Esta SPEC descreve o comportamento **como construído** no PR #3. Nomes de arquivos, o indicador de carregamento e o selo seguem a implementação do Marcos, que foi revisada e aprovada.

---

## 1. Contexto e Objetivo

O back gera, sob demanda e com cache por filme, um parágrafo de IA com a opinião da comunidade sobre o filme ([`SPEC-BACK-09`](../back/SPEC-BACK-09-ia-e-arquetipo.md)). Esta SPEC coloca esse resumo na ficha do filme (`/movies/[id]`), acima da lista de avaliações, com aviso claro de que o texto foi gerado por IA.

> **Spoiler com IA:** pelo [`CR-02`](../cr/CR-02-spoiler-ia-pos-publicacao.md), a análise acontece no back depois de publicar e grava `is_spoiler`, que o front já usa para ocultar. **O front não consulta a IA no modal.** A única mudança de front do CR-02 também fica neste card: o limiar de votos da comunidade passa de 3 para **10** no `ReviewCard` (seção 4.4).

## 2. Escopo

- **Inclui:** service, hook e componente do resumo, os estados de exibição, integração na página do filme, mock para desenvolvimento, testes e o limiar de 10 votos de spoiler no `ReviewCard` (CR-02).
- **Não inclui:** gerar ou forçar a regeneração do resumo, resumo por pessoa, outros idiomas, perguntas livres à IA, qualquer consulta de spoiler no modal (rota `POST /api/v1/reviews/spoiler-check`) e selo de origem do spoiler.

## 3. Regras de Negócio

- **RN-01 (Não bloquear a ficha).** O resumo é buscado **no navegador, depois** de a ficha carregar, por um componente independente. A página do filme não espera a IA, e uma falha no resumo nunca quebra nem esconde o resto da ficha.
- **RN-02 (Estados).**

  | Situação | Exibição |
  | :--- | :--- |
  | Carregando | Spinner com o rótulo "Gerando o resumo da comunidade" e a dica "A primeira geração pode levar alguns segundos." |
  | `ready` | Texto, selo "Gerado por IA" e "Gerado em *data e hora*" |
  | `insufficient_reviews` (menos de 3 avaliações elegíveis) | **Nada.** O bloco não aparece |
  | `unavailable`, erro de rede, erro HTTP, resposta inesperada, `ready` sem texto ou tempo esgotado | "O resumo da comunidade está indisponível no momento.", sem cor de erro |

- **RN-03 (Aviso de IA sempre visível).** Sempre que houver texto, o selo **"Gerado por IA"** aparece junto dele. A data de geração aparece quando o back a envia e é válida.
- **RN-04 (Data no horário local).** `generated_at` chega em UTC e é exibido no fuso do navegador, em pt-BR (por exemplo, "Gerado em 23/11/2026, 12:00"). Se a string vier sem fuso, é tratada como UTC. Data inválida simplesmente não é exibida.
- **RN-05 (Texto como dado).** O texto é renderizado como texto, nunca interpretado como HTML ou markdown.
- **RN-06 (Sem spoilers por responsabilidade do back).** O back exclui do resumo as avaliações marcadas como spoiler (pelo autor ou pela IA) e as que têm 10 ou mais votos de spoiler (CR-02). O front não filtra nada.
- **RN-07 (Uma requisição por filme).** Chamadas simultâneas para o mesmo filme compartilham uma única requisição. Isso evita que o modo de desenvolvimento do React, que executa os efeitos duas vezes, faça uma segunda chamada, que receberia `unavailable` do back por haver uma geração em andamento.
- **RN-08 (Tempo máximo).** A espera pelo resumo tem limite de 30 s (`REVIEW_SUMMARY_TIMEOUT_MS`), porque a primeira geração no back pode levar perto de 20 s. Passou disso, o bloco mostra a mensagem de indisponibilidade.
- **RN-09 (Cancelamento).** Ao sair da página antes da resposta, o componente para de esperar, sem cancelar a requisição compartilhada e sem atualizar estado de componente desmontado.
- **RN-10 (Atualização).** O resumo é carregado uma vez por visita à ficha. Publicar uma avaliação **não** recarrega o resumo, porque o back só o regenera depois de algumas avaliações novas.
- **RN-11 (Mock).** Com `NEXT_PUBLIC_USE_API_MOCKS=true`, o service devolve um resumo simulado, com latência de cerca de 1 s: `ready` para filmes com 3 ou mais avaliações mockadas e `insufficient_reviews` para os demais.
- **RN-12 (Limiar de votos, CR-02).** No `ReviewCard`, a ocultação por votos usa a constante `SPOILER_VOTE_THRESHOLD = 10`, no lugar de `counts.spoiler >= 3`. A ocultação por `isSpoiler` não muda.

## 4. Contrato Técnico

### 4.1 Endpoint consumido

`GET /api/v1/movies/{movie_id}/review-summary` (público, sem autenticação). Responde `200` para entrada válida, inclusive em falha da IA.

```json
{
  "status": "ready",
  "text": "A comunidade destaca a direção e a trilha sonora do filme.",
  "generated_at": "2026-11-23T15:00:00Z"
}
```

| Campo | Valores |
| :--- | :--- |
| `status` | `"ready"`, `"insufficient_reviews"` ou `"unavailable"` |
| `text` | texto em `ready`, `null` nos outros |
| `generated_at` | data ISO-8601 em UTC em `ready`, `null` nos outros |

### 4.2 Modelo de domínio (`features/reviews/types.ts`)

```ts
type ReviewSummary =
  | { status: "ready"; text: string; generatedAt: string }
  | { status: "insufficient_reviews" }
  | { status: "unavailable" };
```

O mapper `toReviewSummary` converte qualquer formato inesperado, ou `ready` sem texto, em `unavailable`. O hook expõe à tela os estados `loading`, `hidden` (vindo de `insufficient_reviews`), `unavailable` e `ready`.

### 4.3 Arquivos

Respeita [`docs/arquitetura.md`](https://github.com/logbuster-app/logbuster-web/blob/main/docs/arquitetura.md) e o `AGENTS.md`: sem `fetch` em componente ou hook, service com mapper e testes ao lado do código. O resumo fica na feature `reviews`.

```text
src/features/reviews/
  api/reviewAi.ts                fetchReviewSummary(movieId, signal), toReviewSummary,
                                 requisição compartilhada por filme e limite de 30 s
  api/reviewAi.mock.ts           fetchReviewSummaryMock (RN-11)
  hooks/useReviewSummary.ts      estado do bloco e cancelamento (RN-09)
  components/ReviewSummary.tsx   bloco com os estados da RN-02
  components/ReviewCard.tsx      limiar SPOILER_VOTE_THRESHOLD (RN-12)
  types.ts                       ReviewSummary e SPOILER_VOTE_THRESHOLD
  index.ts                       exporta ReviewSummary, useReviewSummary, fetchReviewSummary
src/shared/lib/format.ts         parseUtcDate e formatLocalDateTime (RN-04)
src/app/movies/[id]/page.tsx     monta <ReviewSummary /> antes de <ReviewList />
```

Na página, o bloco entra dentro do `MovieDetail`, como filho, antes da lista:

```tsx
<MovieDetail movieId={trimmedId} initialMovie={movie}>
  <div className="space-y-6">
    <ReviewSummary movieId={trimmedId} />
    <ReviewList movieId={trimmedId} />
  </div>
</MovieDetail>
```

### 4.4 Limiar de votos no `ReviewCard` (CR-02)

- `ReviewCard.tsx`: `review.isSpoiler || review.counts.spoiler >= SPOILER_VOTE_THRESHOLD`.
- `ReviewCard.test.tsx` cobre 10 votos (oculta) e 9 votos (visível).
- O mock da demo (`reviews.mock.ts`) usa 10 votos na avaliação que deve aparecer protegida.
- Como o perfil (SPEC-FRONT-07) reaproveita o `ReviewCard`, a regra vale para lá também.
- O valor precisa ser igual ao limiar usado pelo back no resumo da comunidade (CR-02).

### 4.5 Visual (identidade "Cartaz do Cinema Novo")

| Elemento | Tratamento |
| :--- | :--- |
| Bloco | `section` com `rounded-card border-2 border-border bg-surface p-5`, cantos retos, sem sombra |
| Título | "Resumo da comunidade" em `font-display text-2xl text-foreground` |
| Carregando | `Spinner` compartilhado e a dica em `text-xs text-muted` |
| Texto | `font-reading max-w-[68ch] text-lg text-foreground` |
| Selo | "Gerado por IA" em chip `border border-border bg-surface-raised`, texto `text-xs font-bold uppercase tracking-wide text-foreground`. Sem ocre, que é exclusivo de spoiler |
| Data | "Gerado em …" em `text-xs text-muted`, dentro de `<time dateTime>` |
| Indisponível | Mesmo bloco, só com o título e a mensagem em `text-sm text-muted`. Sem tom de erro (`danger`) |

Acessibilidade: o bloco é uma `section` com `aria-labelledby` apontando para o título, `aria-busy` enquanto carrega e `aria-live="polite"` para anunciar o texto quando chega.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Resumo pronto
- **Dado** um filme com 3 ou mais avaliações elegíveis e resumo disponível
- **Quando** a pessoa abre a ficha do filme
- **Então** a ficha aparece sem esperar o resumo, o bloco mostra o spinner e, em seguida, o texto, o selo "Gerado por IA" e a data de geração no horário local.

### Cenário 2: Poucas avaliações
- **Dado** um filme com menos de 3 avaliações elegíveis
- **Quando** a ficha é aberta
- **Então** o bloco não aparece.

### Cenário 3: IA indisponível
- **Dado** que o back responde `unavailable`, a rede falha ou a espera passa de 30 s
- **Quando** a ficha é aberta
- **Então** o bloco mostra "O resumo da comunidade está indisponível no momento." sem cor de erro, e o resto da ficha funciona normalmente.

### Cenário 4: Texto com marcação
- **Dado** um resumo cujo texto contém `<b>` ou `**negrito**`
- **Quando** é exibido
- **Então** aparece como texto literal, sem formatação nem HTML interpretado.

### Cenário 5: Sair antes da resposta
- **Dado** a ficha aberta com o resumo ainda carregando
- **Quando** a pessoa volta para a busca
- **Então** o componente para de esperar, sem erro no console nem atualização de estado em componente desmontado.

### Cenário 6: Chamadas simultâneas
- **Dado** duas montagens do bloco para o mesmo filme ao mesmo tempo (por exemplo, o modo estrito do React)
- **Quando** o resumo é buscado
- **Então** apenas uma requisição chega ao back.

### Cenário 7: Limiar de 10 votos
- **Dado** uma avaliação sem marcação de spoiler
- **Quando** tem 9 votos de spoiler
- **Então** o texto aparece normalmente
- **E** com 10 votos, o texto fica oculto com "Mostrar análise".

## 6. Definição de Pronto (DoD)

- [x] Service, mock, hook e componente com testes ao lado do código, cobrindo os estados, o mapper, a requisição compartilhada, o tempo máximo e o cancelamento.
- [x] Limiar de 10 votos no `ReviewCard`, com teste e mock atualizados.
- [x] Consulta de spoiler no modal removida (CR-02).
- [x] `npm run check` e `npm run build` passando localmente.
- [x] Documentação em `logbuster-web/docs/features/resumo-da-comunidade.md`.
- [ ] CI verde no PR #3.
- [ ] Testado contra o deploy de teste (23/11), com a chave da IA configurada.
- [ ] PR #3 mergeado na `main`, depois do CR-02 do back.
- [ ] Status alterado para `Implemented`.
