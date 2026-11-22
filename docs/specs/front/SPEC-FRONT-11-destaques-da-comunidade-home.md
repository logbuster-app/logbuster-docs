# SPEC-FRONT-11 — Destaques da Comunidade na Tela Inicial

- **Status:** Draft — **com corte**
- **Card(s) Trello:** `[FRONT-11]` *(atualizar com o link do card)*
- **Repositório:** `logbuster-web`
- **Responsável:** Marcos Vinicius, com apoio de Marcelo Palmeira (Back A)
- **Dependências:** [`SPEC-BACK-10`](../back/SPEC-BACK-10-ranking-por-destaque.md) (endpoint de ranking), [`SPEC-FRONT-05`](SPEC-FRONT-05-adaptar-busca-para-backend.md) (busca, `MovieCard`, `MovieGrid`), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md)
- **Entrega:** 24/11/2026 ao meio-dia

> **Regra de corte.** Se não estiver funcionando no deploy até **24/11 ao meio-dia**, a seção não vai para a entrega e vira "próximo passo" no pitch. Nada fica pela metade na home.

---

## 1. Contexto e Objetivo

Os destaques das avaliações (Trilha Sonora, Fotografia, Roteiro, Atuação, Direção, Efeitos Especiais) passam a servir para **descobrir filmes**. A tela inicial ganha a seção **"Destaques da comunidade"**: a pessoa escolhe um aspecto e vê os filmes que a comunidade mais destacou nele.

É o principal ponto de diferenciação do produto na primeira tela: descobrir filmes **pela qualidade que a comunidade reconhece**, e não por um algoritmo.

## 2. Escopo

- **Inclui:** a seção na home, os chips dos 6 aspectos, a grade de filmes, o estado na URL, os estados de carregamento, vazio e erro, mock e testes.
- **Não inclui:** combinar a seção com a busca textual, vários aspectos ao mesmo tempo, paginação, filtro de avaliações dentro da ficha do filme e ranking ponderado.

## 3. Regras de Negócio

- **RN-01 (Quando aparece).** A seção aparece na home **quando não há busca digitada** (`/` sem `?q=`), abaixo do campo de busca. Com uma busca ativa, só os resultados da busca aparecem, como hoje.
- **RN-02 (Aspectos).** Seis chips, na ordem oficial e com os nomes em português. Um chip fica sempre selecionado. Ao abrir a home, o padrão é o **primeiro aspecto que tiver filmes**, buscando na ordem oficial. Se nenhum tiver, fica o primeiro (Trilha Sonora) com o estado vazio.
- **RN-03 (Estado na URL).** O aspecto escolhido fica em `?destaque=<valor>` (por exemplo, `/?destaque=soundtrack`), para ser compartilhável e sobreviver ao recarregar, no mesmo padrão do `?q=` da busca. Valor inválido é ignorado e cai no padrão da RN-02.
- **RN-04 (Grade).** Até 10 filmes, reaproveitando o `MovieCard` e a `MovieGrid` da busca. Cada card mostra, além do título e do ano, a legenda **"*N* destaques em *Aspecto*"** (por exemplo, "7 destaques em Trilha Sonora"). Clicar leva à ficha do filme.
- **RN-05 (Estados).**
  - Carregando: esqueleto da grade, sem esconder os chips.
  - Vazio (`items: []`): "Ninguém destacou *Aspecto* ainda. Avalie um filme e seja o primeiro."
  - Erro: `StateMessage` neutro "Não foi possível carregar os destaques agora.", sem afetar a busca.
- **RN-06 (Isolamento da busca).** Uma falha na seção nunca quebra o campo de busca nem os resultados.

## 4. Contrato Técnico

### 4.1 Endpoint consumido

`GET /api/v1/highlights/{highlight}/movies?limit=10` ([`SPEC-BACK-10`](../back/SPEC-BACK-10-ranking-por-destaque.md)). Os itens têm o mesmo formato da busca, mais `highlight_count`.

### 4.2 Modelo de domínio

```ts
type HighlightedMovie = MovieSummary & { highlightCount: number };
```

Reaproveitar o mapper da busca (`toMovieSummary`) e acrescentar `highlightCount`.

### 4.3 Arquivos esperados

Respeita [`docs/arquitetura.md`](https://github.com/logbuster-app/logbuster-web/blob/main/docs/arquitetura.md): a seção é de descoberta de filmes, então fica na feature `movies`.

```text
src/shared/lib/highlights.ts                      HIGHLIGHTS (ordem oficial) e HIGHLIGHT_LABELS
src/features/reviews/types.ts                     passa a reexportar os rótulos de shared (sem quebrar quem já usa)
src/features/movies/
  api/highlights.ts                               fetchMoviesByHighlight(highlight, signal), com mapper
  api/highlights.mock.ts                          mock (NEXT_PUBLIC_USE_API_MOCKS=true)
  hooks/useCommunityHighlights.ts                 aspecto da URL, busca e estados
  components/highlights/CommunityHighlights.tsx   seção com chips e grade
  components/search/MovieCard.tsx                 prop opcional `caption` (RN-04)
src/app/page.tsx                                  monta <CommunityHighlights /> junto da busca
```

**Por que os rótulos sobem para `shared`:** hoje `REVIEW_HIGHLIGHT_LABELS` está em `features/reviews`, e uma feature não pode importar outra. Movê-los para `shared/lib/highlights.ts` permite que `movies` e `reviews` usem a mesma lista. `features/reviews` continua exportando o mesmo nome, para não quebrar nada.

### 4.4 Visual (identidade "Cartaz do Cinema Novo")

| Elemento | Tratamento |
| :--- | :--- |
| Título da seção | "Destaques da comunidade" em `font-display`, com subtítulo em `text-muted`: "Filmes que a comunidade mais elogiou em cada aspecto." |
| Chips | Mesmo padrão do `HighlightPicker`: inativo `border-2 border-border text-muted`, selecionado `border-foreground bg-foreground text-background`, `min-h-11`. Rolagem horizontal **só na faixa de chips** em telas estreitas |
| Grade | `MovieGrid` existente |
| Legenda do card | `text-xs text-muted`, abaixo do ano |

Acessibilidade: os chips formam um grupo de botões alternáveis (`aria-pressed`), navegáveis por teclado. A grade anuncia a troca de aspecto (`aria-live="polite"`). Sem rolagem horizontal da página em 375 px.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Home sem busca
- **Dado** a home aberta sem `?q=`
- **Quando** ela carrega
- **Então** aparece a seção "Destaques da comunidade", com um aspecto selecionado e até 10 filmes com a legenda "*N* destaques em *Aspecto*".

### Cenário 2: Trocar de aspecto
- **Dado** a seção aberta em "Trilha Sonora"
- **Quando** a pessoa escolhe "Fotografia"
- **Então** a URL vira `?destaque=cinematography`, a grade mostra os filmes de Fotografia e, ao recarregar, a escolha permanece.

### Cenário 3: Com busca
- **Dado** uma busca ativa (`/?q=matrix`)
- **Quando** a página é exibida
- **Então** a seção não aparece, e os resultados da busca ficam como hoje.

### Cenário 4: Vazio e erro
- **Dado** um aspecto sem filmes, ou o back fora do ar
- **Quando** é selecionado
- **Então** aparece a mensagem de vazio ou de erro, os chips continuam funcionando e a busca segue normal.

### Cenário 5: Ficha do filme
- **Dado** a grade de um aspecto
- **Quando** a pessoa clica em um filme
- **Então** vai para a ficha `/movies/{id}`.

## 6. Definição de Pronto (DoD)

- [ ] Service, mock, hook e componente com testes, cobrindo mapper, estados, URL e padrão da RN-02.
- [ ] Rótulos de destaque movidos para `shared` sem quebrar `reviews`.
- [ ] Testado contra a API local e o deploy de teste, com dados de demonstração em vários filmes e destaques.
- [ ] `npm run check` e `npm run build` passando no CI.
- [ ] Documentação em `logbuster-web/docs/features/destaques-da-comunidade.md`.
- [ ] Funcionando no deploy até 24/11 ao meio-dia. Caso contrário, a seção fica fora da entrega.
