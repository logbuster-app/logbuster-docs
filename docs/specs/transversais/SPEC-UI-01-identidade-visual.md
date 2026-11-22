# SPEC-UI-01 — Identidade Visual e Sistema de Design do Front-end

- **Status:** Approved (revisão 2, 22/11/2026: identidade **"Cartaz do Cinema Novo"**). A seção 6 é uma proposta para a Parte 2 e está em revisão.
- **Card(s) Trello:** [`LB-008`](https://trello.com/c/m7bgLvRQ) (*Wireframes, protótipo e sistema de design*) + Requisito Transversal de Identidade Visual
- **Repositório:** `logbuster-web` (`src/app/globals.css`, `src/app/layout.tsx` e componentes em `src/shared` / `src/features`)
- **Responsável:** Marcos, Elton e Thyago (Revisão: João Raphael)
- **Dependências:** [`SPEC-FRONT-00`](../front/SPEC-FRONT-00-setup-arquitetura-web.md)
- **Espelho no código:** [`logbuster-web/docs/identidade-visual.md`](https://github.com/logbuster-app/logbuster-web/blob/main/docs/identidade-visual.md). Em caso de divergência, valem os tokens de `src/app/globals.css`, e esta SPEC deve ser corrigida.

> **Histórico.** A revisão 1 descrevia roxo profundo e âmbar sobre grafite, com cantos arredondados. Ela foi substituída em 10/11/2026 (commit `102e42c` do `logbuster-web`) pela identidade "Cartaz do Cinema Novo". Nenhuma tela nova deve usar roxo, âmbar ou cantos arredondados.

---

## 1. Contexto e Objetivo

Oficializar a identidade visual do **LogBuster** que já está implementada no `logbuster-web`: a crítica brasileira de cinema impressa como **cartaz do Cinema Novo**, com **cor chapada, letra condensada pesada e cantos retos**, como recortes de papel.

O argumento é o produto, então o voto de mérito é o elemento visual mais forte de cada análise: o **carimbo** "Bem argumentada". A identidade empresta só quatro coisas desse mundo: tipografia, paleta, densidade e o gesto do carimbo. Navegação e controles continuam seguindo os padrões da web.

## 2. Escopo e Restrições de Marca

- **Inclui:** paleta semântica (`@theme` do Tailwind CSS v4), tipografia, geometria, padrões dos componentes existentes (cabeçalho, cards de filme, ficha do filme, estrelas, chips de destaque, card de avaliação com spoiler, carimbo de mérito, modal) e padrões das telas da Parte 2 (seção 6), além de acessibilidade e contraste.
- **Proibições explícitas (o que NÃO copiar do Letterboxd):** logotipo de três círculos, textos proprietários, paleta original (`#14181C` + verde `#00E054` / azul `#40BCF4`) e clones de layout pixel a pixel.

---

## 3. Regras de Design System

- **RN-01 (Só tokens).** Nenhum `.tsx` usa hex solto ou cor da paleta padrão do Tailwind (`bg-red-600`, `text-gray-400`, `#fff`). Todo componente consome os tokens semânticos de `src/app/globals.css`. A escala `brand-300` a `brand-900` pode ser usada em superfícies decorativas (por exemplo, fundo padrão do perfil). Para mudar a identidade, edita-se só o bloco `@theme`.
- **RN-02 (Contraste do vermelhão).** Sobre `bg-primary` o texto é sempre `text-primary-foreground` (tinta preta, 4,75:1), **nunca** branco ou osso. Para links, acentos e hover em texto sobre `background`/`surface`, usar `text-primary-text` (5,6:1).
- **RN-03 (Papéis fixos das cores de destaque).**
  - **Osso (`secondary` / `foreground`)** é a cor das **estrelas e notas** e do texto principal. Nunca branco puro.
  - **Ocre (`warning`)** é **exclusivo** do aviso de spoiler.
  - **`danger`** (salmão) é só para erro e é distinto do vermelhão da marca. Erro nunca usa `primary`.
- **RN-04 (Geometria de cartaz).** Cantos retos (`--radius-card: 0`, então `rounded-card` resulta em canto reto). **Sem sombras e sem gradientes.** Bordas de 2 px (`border-2`): `border-foreground` para molduras fortes (pôster da ficha, modal, avatar) e `border-border` para as suaves (campos e itens inativos). `rounded-full` não é usado.
- **RN-05 (Estado por forma, não só por cor).** Selecionado, ativo e votado mudam preenchimento, peso ou ícone, e não apenas a cor. Exemplo: chip selecionado vira bloco osso cheio, e voto próprio vira bloco osso com "✓ Votado".
- **RN-06 (Pôsteres).** Proporção fixa `aspect-[2/3]`, fundo `bg-surface-raised` e fallback "Sem poster" (`text-subtle`) quando não há URL ou a imagem falha (`PosterImage`).
- **RN-07 (Acessibilidade e teclado).** `:focus-visible` com contorno de 2 px em `primary-text` (sobre `bg-primary`, usar `focus-visible:outline-primary-foreground`). Alvos de toque de no mínimo 44 px (`min-h-11` / `h-11` / `size-11`) em controles e chips. Respeitar `prefers-reduced-motion`. Sem rolagem horizontal em 375 px.
- **RN-08 (Ícones).** SVG próprio de traço único em `src/shared/components` (por exemplo, `WarningIcon`, `StarIcon`, `CloseIcon`, `PlusIcon`, `CheckIcon`, `ThumbUpIcon`). Sem emojis e sem bibliotecas de ícones externas.

---

## 4. Contrato Técnico de Tokens (`src/app/globals.css`)

### 4.1 Paleta Oficial

| Categoria | Token CSS (`@theme`) | Classe Tailwind | Valor | Aplicação |
| :--- | :--- | :--- | :--- | :--- |
| **Marca** | `--color-primary` | `bg-primary` | `#E8452C` | Faixa do cabeçalho, botões principais (Buscar, Avaliar, Publicar), carimbo de mérito líder |
| **Marca** | `--color-primary-hover` | `hover:bg-primary-hover` | `#F2573E` | Hover de superfícies primárias |
| **Marca** | `--color-primary-foreground` | `text-primary-foreground` | `#14100E` | Tinta preta sobre o vermelhão |
| **Marca** | `--color-primary-text` | `text-primary-text` | `#F26B50` | Links, hover, foco e acentos em texto sobre fundo escuro |
| **Marca (escala)** | `--color-brand-300` … `--color-brand-900` | `bg-brand-800` etc. | `#F8A08E` … `#4A130B` | Superfícies decorativas derivadas da marca |
| **Destaque** | `--color-secondary` | `text-secondary` | `#EDE3CE` | Estrelas preenchidas e notas (osso) |
| **Destaque** | `--color-secondary-foreground` | `text-secondary-foreground` | `#14100E` | Texto escuro sobre superfícies em osso |
| **Superfície** | `--color-background` | `bg-background` | `#14100E` | Fundo da página (preto quente de xilogravura) e fundo de campos |
| **Superfície** | `--color-surface` | `bg-surface` | `#1D1814` | Painéis, modal, aviso de spoiler, `StateMessage` |
| **Superfície** | `--color-surface-raised` | `bg-surface-raised` | `#271F1A` | Placeholder de pôster, esqueletos de carregamento, hover de botões discretos |
| **Superfície** | `--color-border` | `border-border` | `#4D4238` | Divisórias e bordas suaves ou inativas |
| **Texto** | `--color-foreground` | `text-foreground` / `border-foreground` | `#EDE3CE` | Texto principal, molduras fortes, chip selecionado |
| **Texto** | `--color-muted` | `text-muted` | `#B9AD97` | Metadados (ano, data, rótulos auxiliares) |
| **Texto** | `--color-subtle` | `text-subtle` | `#968B78` | Placeholders, estrelas vazias, "Sem poster" |
| **Estado** | `--color-success` | `text-success` | `#4CAF7D` | Confirmação |
| **Estado** | `--color-warning` | `text-warning` / `border-warning` | `#E0A91A` | Aviso de **spoiler** (exclusivo) |
| **Estado** | `--color-danger` | `text-danger` / `border-danger` | `#FF8F80` | Erros e campos obrigatórios (`*`) |
| **Estado** | `--color-info` | `text-info` | `#7FB0FF` | Avisos informativos neutros |

### 4.2 Tipografia

Fontes hospedadas pelo `next/font` em `src/app/layout.tsx` (baixadas no build, sem requisição externa no navegador).

| Papel | Fonte | Classe | Uso |
| :--- | :--- | :--- | :--- |
| Títulos e rótulos de destaque | **Archivo** em largura 62% e peso 900, caixa alta | `font-display` | Logotipo, títulos de página e seção, números grandes (nota, contagem de votos) |
| Interface | **Archivo** normal | padrão (`font-sans`) | Botões, rótulos, metadados |
| Leitura | **Source Serif 4** | `font-reading` | Sinopse e texto das análises, com medida de até `max-w-[68ch]` |

Padrões de uso:
- Título de página: `font-display text-5xl sm:text-7xl text-foreground` (ficha do filme) ou `text-3xl`/`text-4xl` em telas menores, como modal e formulários.
- Rótulo de campo ou seção pequena: `text-xs font-bold uppercase tracking-wide text-foreground`.
- Botões: `text-sm font-bold uppercase tracking-wide`.
- Metadados: `text-xs sm:text-sm text-muted`.

### 4.3 Layout

- Container global: `mx-auto w-full max-w-6xl px-4 sm:px-6` (aplicado em `RootLayout` e `SiteHeader`).
- Cabeçalho: faixa `bg-primary` cheia, `h-16`, logotipo em `font-display text-4xl`.

---

## 5. Padrões dos Componentes Existentes

1. **Cabeçalho (`SiteHeader`).** Faixa vermelha cheia, como a tarja de um cartaz. Texto em `text-primary-foreground` e foco em `outline-primary-foreground`.
2. **Card de filme (`MovieCard`).** Pôster `aspect-[2/3] border-2 border-border bg-surface-raised`, com hover da borda em `primary-text`. Título `text-sm font-bold uppercase tracking-wide` e ano em `text-muted`.
3. **Ficha do filme (`MovieDetail`).** Duas colunas no desktop (`grid-cols-1 md:grid-cols-[280px_1fr] gap-10`). Pôster com moldura forte (`border-2 border-foreground`). Título em `font-display`. **Nota LogBuster:** bloco osso com a média em display de 72 px, seguida de "/ 5" em display de 36 px, com rótulo e contagem ao lado. Sem avaliações, exibe "Sem avaliações".
4. **Estrelas (`StarRating` / `StarRatingInput`).** Cinco `StarIcon` em `text-secondary` (osso), com meia estrela por recorte e vazias em `text-subtle`, acompanhadas do valor numérico (ex.: `4,5 / 5`).
5. **Chips de destaque (`HighlightPicker`).** `min-h-11 rounded-card border-2 px-3 text-sm font-semibold`. Inativo: `border-border text-muted`, com hover para `foreground`. Selecionado (até 2): bloco cheio `border-foreground bg-foreground text-background`. Desabilitado: `opacity-45`.
6. **Card de avaliação e spoiler (`ReviewCard`).** Texto em `font-reading max-w-[68ch] text-lg`. Spoiler não revelado: caixa `border-2 border-warning bg-surface p-4`, com `WarningIcon` e "Esta avaliação contém spoiler" em `text-warning font-bold uppercase`, e o link **"Mostrar análise"** (`text-primary-text underline`).
7. **Carimbo de mérito ("Bem argumentada").** Bloco com a contagem em letra grande ("15 votos") e uma faixa de ação embaixo ("+ Votar" ou "✓ Votado"). Análise líder da ordenação por mérito: bloco vermelho cheio. Voto do próprio visitante: bloco osso. Demais: só contorno. **Curtir** e **Spoiler** têm ícone, contorno osso e contador separado por filete.
8. **Modal (`ReviewModal` + `useModalBehavior`).** Fundo `bg-background/90`. Caixa `border-2 border-foreground bg-surface`, com cabeçalho separado por `border-b-2 border-foreground` e título em `font-display text-3xl`. Fecha por `Escape`, "Cancelar" ou "×", nunca por clique fora. Pede confirmação antes de descartar texto digitado.
9. **Mensagens de estado (`StateMessage`).** `border-2 bg-surface p-6 text-center`, com título em `font-display text-2xl`. No tom `danger`, usa borda e título em `danger` e `role="alert"`.

---

## 6. Padrões das Telas da Parte 2 (proposta, em revisão)

Padrões para as telas novas, montados só com os tokens e componentes acima. Servem para que [`SPEC-FRONT-07`](../front/SPEC-FRONT-07-contas-sessao-perfil.md) e as próximas SPECs de front fiquem visualmente coerentes.

1. **Formulários (cadastro, login, editor de perfil).** Mesmo padrão do `ReviewModal`:
   - Rótulo `text-xs font-bold uppercase tracking-wide text-foreground`, com obrigatório marcado por `*` em `text-danger`.
   - Campo `h-11 w-full rounded-card border-2 border-border bg-background px-3 text-sm`, com foco `focus-visible:border-foreground` e placeholder `text-subtle`.
   - Dica ou contador em `text-xs text-muted`.
   - Erro geral em caixa `role="alert" border-2 border-danger bg-background p-3 text-sm text-danger`, acima dos campos.
2. **Botões.**
   - Primário: `min-h-11 bg-primary px-5 text-primary-foreground hover:bg-primary-hover font-bold uppercase tracking-wide`.
   - Secundário: `min-h-11 border-2 border-border px-5 text-muted hover:border-foreground hover:text-foreground`.
   - Carregando: texto no gerúndio ("Entrando...", "Salvando...") e `disabled`.
3. **Menu de conta no cabeçalho.** Sobre a faixa vermelha: links "Entrar" e "Criar conta" em `text-primary-foreground font-bold uppercase`. Logado: avatar pequeno (`size-9`, moldura `border-2 border-primary-foreground`), nome e "Sair".
4. **Perfil público.**
   - Banner em 16:9 com altura máxima moderada e `object-cover`. Sem imagem, faixa chapada `bg-brand-800`.
   - Avatar quadrado `size-24 sm:size-32 border-2 border-foreground`, sobreposto à base do banner.
   - Nome em `font-display`, data de entrada em `text-muted` e bio em `font-reading`.
   - Avatar sem imagem: inicial do nome em `font-display` sobre `bg-surface-raised`.
5. **Conteúdo gerado por IA (resumo da comunidade e avisos).** Bloco `border-2 border-border bg-surface p-4`, com selo **"Gerado por IA"** em `text-xs font-bold uppercase tracking-wide text-info` e data em `text-muted`. Texto em `font-reading`. Indisponível: `StateMessage` neutro, sem tom de erro.
6. **Aviso de spoiler sugerido por IA (no modal).** Reaproveita a linguagem do spoiler: caixa `border-2 border-warning bg-background p-4` com `WarningIcon`, título em `text-warning` e duas ações (primária e secundária).
7. **Arquétipo do cinéfilo.** Bloco osso, na mesma linguagem da Nota LogBuster: título do arquétipo em `font-display` sobre `bg-foreground text-background` e explicação curta em `text-sm`. Sem arquétipo: texto `text-muted` explicando o mínimo de avaliações.
8. **Estados vazios, carregamento e erro.** `StateMessage` para vazio e erro. Esqueletos em `bg-surface-raised` com `animate-pulse`, respeitando `prefers-reduced-motion`.

---

## 7. Critérios de Aceite (Gherkin)

### Cenário 1: Consistência de tokens sem cores soltas
- **Dado** qualquer componente novo ou alterado em `src/features` ou `src/shared`
- **Quando** revisado no PR
- **Então** todas as classes de cor pertencem aos tokens de `globals.css`, não há `rounded-*` diferente de `rounded-card`, nem sombras ou gradientes.

### Cenário 2: Papéis das cores respeitados
- **Dado** uma tela com nota, spoiler e erro ao mesmo tempo
- **Quando** exibida
- **Então** estrelas e nota usam osso, o spoiler usa ocre e o erro usa `danger`, sem trocar os papéis entre si nem usar o vermelhão da marca para erro.

### Cenário 3: Responsividade e navegação por teclado
- **Dado** uma tela aberta em 375 px ou 1280 px
- **Quando** a pessoa navega com `Tab` por busca, ficha, modal, votos, formulários e perfil
- **Então** não há rolagem horizontal, todos os focos ficam visíveis e os alvos de toque têm no mínimo 44 px.

## 8. Definição de Pronto (DoD)

- [x] `globals.css` com os tokens da identidade "Cartaz do Cinema Novo".
- [x] `logbuster-web/docs/identidade-visual.md` descrevendo a identidade atual.
- [ ] Seção 6 revisada e aprovada pela equipe de front (Thyago, Marcos e Davi).
- [ ] Card `LB-008` atualizado com o link desta revisão.
