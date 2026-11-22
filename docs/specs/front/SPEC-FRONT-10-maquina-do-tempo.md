# SPEC-FRONT-10 — Máquina do Tempo (retrospectiva do cinéfilo)

- **Status:** Draft — **condicional**
- **Card(s) Trello:** `[FRONT-10]` *(atualizar com o link do card)*
- **Repositório:** `logbuster-web`
- **Responsável:** Davi
- **Dependências:** [`SPEC-FRONT-09`](SPEC-FRONT-09-perfil-publico-arquetipo.md) (página de perfil, `fetchUserReviews`, `MovieTitleLink`), [`SPEC-BACK-08`](../back/SPEC-BACK-08-auth-perfil-publico.md) (histórico público), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md)
- **Entrega:** congelamento em 24/11/2026

> **Card condicional.** Só começa quando o FRONT-09 estiver em revisão. Se não estiver funcionando no deploy até **24/11 ao meio-dia**, o botão não vai para a entrega e a funcionalidade vira "próximo passo" no pitch. Não pode atrasar o FRONT-09.

> **Sem back.** A [`SPEC-BACK-09`](../back/SPEC-BACK-09-ia-e-arquetipo.md) previa um endpoint opcional (`GET /api/v1/users/me/time-machine`), que não foi implementado. Esta SPEC calcula tudo no front, a partir do histórico público da conta. Nenhuma mudança no back.

---

## 1. Contexto e Objetivo

Uma retrospectiva no estilo "Wrapped": a partir das avaliações de uma conta, mostrar em sequência de cartazes quantos filmes a pessoa avaliou, a nota média, o aspecto que ela mais valoriza, o filme que ela mais amou e, por fim, o arquétipo do cinéfilo. É uma funcionalidade de encantamento, pensada para o pitch e para reforçar a identidade de cinéfilo do produto.

## 2. Escopo

- **Inclui:** botão no perfil, rota da retrospectiva, cálculo das estatísticas no front, sequência de cartazes e estados de carregamento, vazio e erro.
- **Não inclui:** escolha de período, comparação com amigos, compartilhamento como imagem, endpoint no back, animações complexas.

## 3. Regras de Negócio

- **RN-01 (Onde aparece).** No perfil (`/users/[id]`), um botão **"Máquina do Tempo"** leva para `/users/[id]/retrospectiva`. A retrospectiva é pública, como o perfil, e o título diz de quem é ("A jornada de *nome*").
- **RN-02 (Mínimo).** O botão só aparece com **3 ou mais** avaliações vinculadas à conta, o mesmo mínimo do arquétipo. Acessando a rota direto com menos, mostra "A Máquina do Tempo aparece a partir de 3 avaliações." e um link de volta ao perfil.
- **RN-03 (Período).** Todo o histórico da conta. Não há seleção de período.
- **RN-04 (Fonte dos dados).** Buscar as avaliações por `GET /api/v1/users/{user_id}/reviews`, com `page_size=50`, percorrendo as páginas até o `total`, com teto de **4 páginas (200 avaliações)**. Acima disso, a retrospectiva usa as 200 mais recentes, e isso é aceito.
- **RN-05 (Estatísticas).** Calculadas por uma função pura, testável sem rede:

  | Cartaz | Cálculo | Desempate |
  | :--- | :--- | :--- |
  | **Filmes avaliados** | Quantidade de `movie_id` distintos | — |
  | **Nota média** | Média de `rating` de todas as avaliações, com uma casa decimal (formato pt-BR, por exemplo "4,2") | — |
  | **O que você mais valoriza** | Destaque (`highlights`) mais escolhido, com o nome em português e a contagem | Ordem oficial dos destaques (Trilha Sonora, Fotografia, Roteiro, Atuação, Direção, Efeitos Especiais) |
  | **O filme que você mais amou** | Avaliação com a maior `rating`, mostrando título (via `MovieTitleLink`) e nota | A mais recente |
  | **Seu arquétipo** | `archetype` do perfil (`{ title, description }`) | — |

  Se nenhuma avaliação tiver destaque, o cartaz de destaques é omitido. Se o arquétipo vier `null`, o cartaz final é omitido.
- **RN-06 (Spoiler).** A retrospectiva **nunca** mostra o texto das avaliações, só números, nomes de aspectos e títulos. Não há risco de spoiler.
- **RN-07 (Navegação).** Os cartazes aparecem um por vez, com botões **"Próximo"** e **"Anterior"**, setas do teclado e indicador de progresso ("2 de 5"). No último, o botão vira **"Voltar ao perfil"**. Com `prefers-reduced-motion`, a troca é sem animação.
- **RN-08 (Falhas).** Erro ao carregar o perfil ou o histórico mostra `StateMessage` com "Não foi possível montar a retrospectiva." e um link de volta ao perfil. Falha ao buscar o título de um filme mostra o `movie_id` como link (mesma regra da SPEC-FRONT-09).

## 4. Contrato Técnico

### 4.1 Arquivos esperados

Respeita [`docs/arquitetura.md`](https://github.com/logbuster-app/logbuster-web/blob/main/docs/arquitetura.md). Reaproveita o que a SPEC-FRONT-09 cria.

```text
src/
  app/users/[id]/retrospectiva/page.tsx   Compõe profile + reviews + movies
  features/reviews/
    timeMachine.ts                        computeTimeMachine(reviews) — função pura (RN-05)
    timeMachine.test.ts
    hooks/useUserReviewsAll.ts            percorre as páginas com teto (RN-04)
    components/TimeMachine.tsx            sequência de cartazes (RN-07)
  features/profile/components/ProfileHeader.tsx   botão "Máquina do Tempo" (RN-01, RN-02)
```

Modelo calculado:

```ts
type TimeMachineStats = {
  ratedMoviesCount: number;
  ratingAvg: number;
  topHighlight: { highlight: ReviewHighlight; count: number } | null;
  topMovie: { movieId: string; rating: number };
};
```

### 4.2 Endpoints consumidos (já existentes)

| Uso | Chamada |
| :--- | :--- |
| Nome e arquétipo | `GET /api/v1/users/{user_id}` |
| Avaliações | `GET /api/v1/users/{user_id}/reviews?page=&page_size=50` |
| Título do filme | Serviço de detalhe de filme já existente, via `MovieTitleLink` |

### 4.3 Visual (identidade "Cartaz do Cinema Novo")

Cada estatística é um **cartaz**: bloco grande de cor chapada, número em `font-display` muito grande e rótulo curto em caixa alta, como a Nota LogBuster da ficha do filme. Alternar o fundo entre `bg-primary` (com `text-primary-foreground`), `bg-foreground` (com `text-background`) e `bg-surface` com `border-2 border-foreground`. O cartaz do arquétipo segue o bloco osso da SPEC-UI-01, seção 6, item 7. Cantos retos, sem sombra e sem gradiente.

Acessibilidade: a região dos cartazes é anunciada (`aria-live="polite"`), os botões têm 44 px, o indicador de progresso é texto e a navegação funciona só com teclado.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Retrospectiva completa
- **Dado** uma conta com 7 avaliações em 6 filmes, com destaques e arquétipo
- **Quando** alguém abre o perfil e clica em "Máquina do Tempo"
- **Então** vê, em sequência, 6 filmes avaliados, a nota média, o destaque mais escolhido, o filme mais bem avaliado com título e o arquétipo.

### Cenário 2: Desempates
- **Dado** duas avaliações com a nota máxima e dois destaques empatados
- **Quando** a retrospectiva é calculada
- **Então** o filme mais amado é o da avaliação mais recente, e o destaque segue a ordem oficial.

### Cenário 3: Poucas avaliações
- **Dado** uma conta com 2 avaliações
- **Quando** o perfil é aberto
- **Então** o botão "Máquina do Tempo" não aparece
- **E** acessar a rota direto mostra a mensagem do mínimo e o link de volta.

### Cenário 4: Sem destaques ou sem arquétipo
- **Dado** uma conta cujas avaliações não têm destaques, ou cujo arquétipo veio `null`
- **Quando** a retrospectiva é aberta
- **Então** os cartazes correspondentes são omitidos, sem espaço vazio.

### Cenário 5: Falha
- **Dado** que o histórico não carrega
- **Quando** a retrospectiva é aberta
- **Então** aparece "Não foi possível montar a retrospectiva." com link de volta ao perfil.

### Cenário 6: Teclado
- **Dado** a retrospectiva aberta
- **Quando** a pessoa usa as setas e o `Tab`
- **Então** navega entre os cartazes e os botões sem mouse.

## 6. Definição de Pronto (DoD)

- [ ] `computeTimeMachine` com testes cobrindo contagem, média, desempates e omissões.
- [ ] Botão no perfil e rota da retrospectiva funcionando contra a API local e o deploy de teste.
- [ ] Conta de demonstração com dados que mostrem todos os cartazes (pode ser a mesma do FRONT-09).
- [ ] `npm run check` e `npm run build` passando no CI.
- [ ] Documentação em `logbuster-web/docs/features/maquina-do-tempo.md`.
- [ ] Funcionando no deploy até 24/11 ao meio-dia. Caso contrário, o botão fica fora da entrega.
