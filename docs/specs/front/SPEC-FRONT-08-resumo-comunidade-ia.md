# SPEC-FRONT-08 — Resumo da Comunidade com IA na Ficha do Filme

- **Status:** Draft
- **Card(s) Trello:** `[FRONT-08]` *(atualizar com o link do card)*
- **Repositório:** `logbuster-web`
- **Responsável:** Marcos Vinicius, com apoio de João Victor (Back B)
- **Dependências:** [`SPEC-BACK-09`](../back/SPEC-BACK-09-ia-e-arquetipo.md) (resumo da comunidade), [`SPEC-FRONT-06`](SPEC-FRONT-06-adaptar-detalhe-filme-backend.md) (ficha do filme), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md) (seção 6, item 5), [`CR-02`](../cr/CR-02-spoiler-ia-pos-publicacao.md)
- **Entrega:** congelamento em 24/11/2026

---

## 1. Contexto e Objetivo

O back já gera, sob demanda e com cache por filme, um parágrafo de IA com a opinião da comunidade sobre o filme ([`SPEC-BACK-09`](../back/SPEC-BACK-09-ia-e-arquetipo.md)). Esta SPEC coloca esse resumo na ficha do filme (`/movies/[id]`), no topo da seção de avaliações, com aviso claro de que o texto foi gerado por IA.

> **Spoiler com IA não está nesta SPEC.** Pelo [`CR-02`](../cr/CR-02-spoiler-ia-pos-publicacao.md), a análise de spoiler acontece no back depois de publicar e grava `is_spoiler`, que o front já usa para ocultar. Não há aviso no modal e nada muda no front para isso.

## 2. Escopo

- **Inclui:** service, hook e componente do resumo, os quatro estados de exibição (carregando, pronto, indisponível e oculto), integração na página do filme, mock para desenvolvimento e testes.
- **Não inclui:** gerar ou forçar a regeneração do resumo, resumo por pessoa, outros idiomas, perguntas livres à IA, aviso de spoiler no modal, selo de origem do spoiler e qualquer mudança na ocultação de avaliações.

## 3. Regras de Negócio

- **RN-01 (Não bloquear a ficha).** O resumo é buscado **no navegador, depois** de a ficha carregar. A página do filme não espera a IA, e uma falha no resumo nunca quebra nem esconde o resto da ficha.
- **RN-02 (Estados).** A resposta do back tem um `status`, e cada um tem uma exibição:

  | `status` do back | Exibição |
  | :--- | :--- |
  | carregando (antes da resposta) | Esqueleto do bloco, com título e linhas cinzas |
  | `ready` | Bloco com título, texto, selo "Gerado por IA" e data |
  | `insufficient_reviews` (menos de 3 avaliações elegíveis) | **Nada.** O bloco não aparece, sem espaço vazio |
  | `unavailable` (falha, demora ou sem texto elegível) | Bloco discreto: "Resumo indisponível no momento." |
  | erro de rede ou resposta inesperada | Mesmo tratamento de `unavailable` |

- **RN-03 (Aviso de IA sempre visível).** Sempre que houver texto, o selo **"Gerado por IA"** e a data de geração ("Gerado por IA em 23 de nov. de 2026") aparecem junto dele, nunca escondidos atrás de um clique.
- **RN-04 (Texto como dado).** O texto é exibido como texto puro, preservando quebras de linha. Nunca interpretar HTML ou markdown (nada de `dangerouslySetInnerHTML`).
- **RN-05 (Sem spoilers por responsabilidade do back).** O back já exclui do resumo as avaliações marcadas como spoiler (pelo autor ou pela IA) e as que têm 3 ou mais votos de spoiler. O front não filtra nada e não oculta o resumo.
- **RN-06 (Atualização).** O resumo é carregado uma vez por visita à ficha. Publicar uma avaliação **não** recarrega o resumo, porque o back só o regenera depois de algumas avaliações novas. Ao sair da página antes da resposta, a requisição é cancelada.
- **RN-07 (Mock).** Com `NEXT_PUBLIC_USE_API_MOCKS=true`, o service devolve um resumo simulado, com latência, permitindo testar os quatro estados sem a IA real, no mesmo padrão de `reviews.mock.ts`.

## 4. Contrato Técnico

### 4.1 Endpoint consumido

`GET /api/v1/movies/{movie_id}/review-summary` (público, sem autenticação). Sempre responde `200`, inclusive em falha da IA.

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

A primeira consulta de um filme pode levar alguns segundos, porque o back chama a IA quando o cache está vencido. O back já aplica timeout e responde `unavailable` se a IA demorar.

### 4.2 Modelo de domínio (camelCase, via mapper)

```ts
type CommunitySummary =
  | { status: "ready"; text: string; generatedAt: string }
  | { status: "insufficient_reviews" }
  | { status: "unavailable" };
```

Valor de `status` desconhecido vira `unavailable`.

### 4.3 Arquivos esperados

Respeita [`docs/arquitetura.md`](https://github.com/logbuster-app/logbuster-web/blob/main/docs/arquitetura.md) e o `AGENTS.md`: sem `fetch` em componente ou hook, service com mapper, testes ao lado do código. O resumo fica na feature `reviews`, porque é derivado das avaliações e é exibido junto delas.

```text
src/features/reviews/
  api/summary.ts                    fetchCommunitySummary(movieId, signal) com mapper
  api/summary.mock.ts               resumo simulado (RN-07)
  hooks/useCommunitySummary.ts      estado do bloco e cancelamento (RN-06)
  components/CommunitySummary.tsx   bloco com os estados da RN-02
  index.ts                          exportar CommunitySummary
src/app/movies/[id]/page.tsx        montar <CommunitySummary /> antes de <ReviewList />
```

Na página, o bloco entra dentro do `MovieDetail`, como filho, antes da lista:

```tsx
<MovieDetail movieId={trimmedId} initialMovie={movie}>
  <CommunitySummary movieId={trimmedId} />
  <ReviewList movieId={trimmedId} />
</MovieDetail>
```

### 4.4 Visual (identidade "Cartaz do Cinema Novo")

Segue a [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md), seção 6, item 5.

| Elemento | Tratamento |
| :--- | :--- |
| Bloco | `border-2 border-border bg-surface p-4`, cantos retos, sem sombra |
| Título | "Resumo da comunidade" em `font-display text-2xl text-foreground` |
| Selo | "Gerado por IA" em `text-xs font-bold uppercase tracking-wide text-info` |
| Data | `text-xs text-muted`, formatada em pt-BR pelo utilitário de `shared/lib/format` |
| Texto | `font-reading max-w-[68ch] text-base text-foreground`, com quebras de linha preservadas |
| Esqueleto | Linhas em `bg-surface-raised` com `animate-pulse`, respeitando `prefers-reduced-motion` |
| Indisponível | Mesmo bloco, só com o título e "Resumo indisponível no momento." em `text-sm text-muted`. Sem tom de erro (`danger`) |

Acessibilidade: o bloco é uma `section` com `aria-labelledby` apontando para o título. O texto carregado é anunciado com `aria-live="polite"`, e o esqueleto tem `aria-busy="true"`.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Resumo pronto
- **Dado** um filme com 3 ou mais avaliações elegíveis e resumo disponível
- **Quando** a pessoa abre a ficha do filme
- **Então** a ficha aparece sem esperar o resumo, o esqueleto aparece no topo das avaliações e, em seguida, o bloco mostra o texto, o selo "Gerado por IA" e a data.

### Cenário 2: Poucas avaliações
- **Dado** um filme com menos de 3 avaliações elegíveis
- **Quando** a ficha é aberta
- **Então** o bloco não aparece e não sobra espaço vazio.

### Cenário 3: IA indisponível
- **Dado** que o back responde `unavailable`, ou a rede falha
- **Quando** a ficha é aberta
- **Então** o bloco mostra "Resumo indisponível no momento." sem cor de erro, e o resto da ficha funciona normalmente.

### Cenário 4: Texto com marcação
- **Dado** um resumo cujo texto contém `<b>` ou `**negrito**`
- **Quando** é exibido
- **Então** aparece como texto literal, sem formatação nem HTML interpretado.

### Cenário 5: Sair antes da resposta
- **Dado** a ficha aberta com o resumo ainda carregando
- **Quando** a pessoa volta para a busca
- **Então** a requisição é cancelada, sem erro no console nem atualização de estado em componente desmontado.

## 6. Definição de Pronto (DoD)

- [ ] Service, mock, hook e componente com testes ao lado do código, cobrindo os quatro estados, o mapper e o cancelamento.
- [ ] Bloco integrado na ficha do filme e testado contra a API local e o deploy de teste (23/11), com a chave da IA configurada.
- [ ] `npm run check` e `npm run build` passando no CI.
- [ ] Documentação criada em `logbuster-web/docs/features/resumo-da-comunidade.md`.
- [ ] SPEC e card atualizados com os links, e status alterado de `Draft` para `Approved` após revisão.
