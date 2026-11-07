# SPEC-UI-01 — Identidade Visual e Sistema de Design do Front-end

- **Status:** Approved
- **Card(s) Trello:** [`LB-008`](https://trello.com/c/m7bgLvRQ) (*Wireframes, protótipo e sistema de design*) + Requisito Transversal de Identidade Visual
- **Repositório:** `logbuster-web` (`src/app/globals.css` e componentes em `src/shared` / `src/features`)
- **Responsável:** Marcos, Elton e Thyago (Revisão: João Raphael)
- **Dependências:** [`SPEC-FRONT-00`](../front/SPEC-FRONT-00-setup-arquitetura-web.md)

---

## 1. Contexto e Objetivo
Oficializar em SPEC SDD a identidade visual do **LogBuster** a partir da base tokenizada já iniciada em `logbuster-web/src/app/globals.css`. A experiência visual é inspirada na clareza editorial e densidade cinematográfica do **Letterboxd** (foco em pôsteres `2:3`, fundo escuro imersivo e leitura confortável de críticas), porém com **identidade de marca própria** construída sobre **roxo profundo** (ações/marca) e **âmbar dourado** (notas/estrelas).

## 2. Escopo e Restrições de Marca
- **Inclui:** Paleta semântica completa (`@theme` do Tailwind CSS v4), tipografia, geometria/espaçamento, anatomia visual dos componentes do Épico de Filmes (Header, Cards de Filme, Ficha do Filme, Estrelas `0,5–5,0`, Chips de Destaques, Card de Avaliação com Spoiler Ocultável e Botões de Voto Qualificado) e regras de acessibilidade/contraste.
- **Proibições explícitas (O que NÃO copiar do Letterboxd):** Logotipos (três círculos laranja/verde/azul), textos proprietários, paleta característica `#14181C` + verde `#00E054` / azul `#40BCF4` ou clones pixel-a-pixel de layout.

---

## 3. Regras de Design System
- **RN-01 (Proibido Hex/Cores Soltas em Componentes):** Nenhum arquivo `.tsx` pode usar classes arbitrárias de cor (`bg-purple-700`, `text-gray-400`, `#fff`). Todo componente consome exclusivamente os tokens semânticos definidos em `src/app/globals.css`.
- **RN-02 (Contraste em Fundo Escuro):** O token `bg-primary` (`#5B2A86`) deve ser usado como superfície de botão com texto `text-primary-foreground` (`#FFFFFF`). Para links, bordas de hover, ícones ativos e acentos textuais sobre `background`/`surface`, é **obrigatório** usar `text-primary-text` (`#A77BD6`) para manter contraste WCAG AA (`>= 4.5:1`).
- **RN-03 (Semântica do Âmbar `secondary`):** O token `secondary` (`#F5B544`) é reservado para **notas médias, estrelas (`0,5` a `5,0`) e selo de mérito ("Bem argumentada")**, criando hierarquia visual imediata entre ações de navegação (roxo) e avaliação cinematográfica (dourado).
- **RN-04 (Pôsteres e Fallback):** Todo pôster de filme respeita proporção fixa `aspect-[2/3]` com `bg-surface-raised`, borda sutil `border-border` e estado placeholder centralizado (`"Sem poster"`, `text-subtle`) quando `posterUrl === null`.
- **RN-05 (Acessibilidade e Teclado):** Todo elemento interativo exibe `:focus-visible` com `outline: 2px solid var(--color-primary-text)` e altura mínima de toque de `44px` (`h-11`) em controles primários (inputs, botões principais) e `36px` (`h-9`) em chips/ações secundárias.

---

## 4. Contrato Técnico de Tokens (`src/app/globals.css`)

### 4.1 Paleta Oficial

| Categoria | Token CSS (`@theme`) | Classe Tailwind | Valor Hex | Aplicação Obrigatória |
| :--- | :--- | :--- | :--- | :--- |
| **Marca** | `--color-primary` | `bg-primary` | `#5B2A86` | Botão primário (Buscar, Publicar avaliação, Avaliar filme) |
| **Marca** | `--color-primary-hover` | `hover:bg-primary-hover` | `#8049B8` | Estado hover de botões primários |
| **Marca** | `--color-primary-foreground` | `text-primary-foreground` | `#FFFFFF` | Texto sobre `bg-primary` |
| **Marca** | `--color-primary-text` | `text-primary-text` / `border-primary-text` | `#A77BD6` | Links, foco, borda ativa de chips/cards e votos marcados |
| **Destaque** | `--color-secondary` | `text-secondary` / `bg-secondary` | `#F5B544` | Estrelas preenchidas, nota média do filme e destaque de mérito |
| **Destaque** | `--color-secondary-foreground` | `text-secondary-foreground` | `#1A1205` | Texto escuro sobre badge sólida `bg-secondary` |
| **Superfície** | `--color-background` | `bg-background` | `#0E0E13` | Fundo global do `<body>` e backdrop base |
| **Superfície** | `--color-surface` | `bg-surface` | `#16161D` | Header, cards de filme, cards de avaliação, inputs e modal |
| **Superfície** | `--color-surface-raised` | `bg-surface-raised` | `#1E1E28` | Botões secundários, chips inativos, skeleton/placeholder de pôster |
| **Superfície** | `--color-border` | `border-border` | `#2A2A37` | Divisórias, contornos de cards, inputs e modais |
| **Texto** | `--color-foreground` | `text-foreground` | `#ECECF3` | Títulos, corpo de avaliações e valores de formulário |
| **Texto** | `--color-muted` | `text-muted` | `#A3A3B5` | Subtítulos, metadados (ano, duração, diretor, data) |
| **Texto** | `--color-subtle` | `text-subtle` | `#7A7A8C` | Placeholders de inputs e dicas auxiliares |
| **Estado** | `--color-success` | `text-success` / `border-success` | `#34B27B` | Confirmação de publicação |
| **Estado** | `--color-warning` | `text-warning` / `border-warning` | `#E8A33D` | Badge/alerta de conteúdo com **Spoiler** |
| **Estado** | `--color-danger` | `text-danger` / `border-danger` | `#E5484D` | Mensagens de erro (`StateMessage tone="danger"`), validações |
| **Estado** | `--color-info` | `text-info` | `#6E9BF5` | Avisos informativos neutros |

### 4.2 Tipografia, Bordas e Layout
- **Tipografia (`--font-sans`):** Pilha sem serifa nativa de alta legibilidade (`ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif`), sem dependência de rede externa.
  - Título de página (`h1`): `text-2xl sm:text-3xl font-bold tracking-tight text-foreground`
  - Título de seção (`h2`): `text-lg sm:text-xl font-semibold text-foreground`
  - Corpo de análise: `text-sm sm:text-base leading-relaxed text-foreground`
  - Metadados / legendas: `text-xs sm:text-sm text-muted`
- **Raio de borda (`--radius-card`):** `0.75rem` (`rounded-card`) para cards, inputs, botões e modal; `rounded-full` permitido para badges/chips de pontos de destaque.
- **Container Global:** `mx-auto w-full max-w-6xl px-4` (já aplicado em `RootLayout` e `SiteHeader`).

---

## 5. Padrões Estruturais dos Componentes do Épico de Filmes (`LB-008`)

1. **Ficha do Filme (`FRONT-02` / `FRONT-06`):**
   - Layout responsivo em 2 colunas no desktop (`grid-cols-1 md:grid-cols-[240px_1fr] gap-8`): pôster `aspect-[2/3]` à esquerda; título, ano, duração, gêneros, direção, elenco, sinopse, bloco de **Nota Média LogBuster** (`text-secondary`, exibindo `"Sem avaliações"` quando `ratingCount === 0`) e botão primário **"Avaliar filme"** à direita.
2. **Seletor e Exibição de Estrelas (`0,5` a `5,0`):**
   - 5 ícones de estrela renderizados em `text-secondary` (cheia, meia estrela e vazia em `text-subtle`), acompanhados do valor numérico formatado (ex.: `4,5 / 5`) para clareza acessível.
3. **Chips de Pontos de Destaque (`highlights`):**
   - Pílulas (`rounded-full border px-3 py-1 text-xs font-medium`). Inativo: `border-border bg-surface-raised text-muted`. Selecionado (até 2): `border-primary-text bg-primary/25 text-foreground`.
4. **Card de Avaliação & Proteção de Spoiler (`FRONT-03`):**
   - Superfície `bg-surface border border-border rounded-card p-4 space-y-3`.
   - Quando marcada como spoiler e ainda não revelada localmente: o parágrafo da análise fica oculto/desfocado atrás de um box `border-warning/40 bg-surface-raised p-4` com aviso `text-warning` (*"Contém spoiler"*) e botão **"Mostrar análise"**.
5. **Barra de Interações Qualificadas (`FRONT-03`):**
   - Três botões compactos lado a lado com contador visível:
     - **Bem argumentada** (`well_argued`) — destaque principal de ordenação; quando ativo pelo visitante, usa `border-primary-text bg-primary/20 text-primary-text`.
     - **Curtir** (`like`) — quando ativo, `border-primary-text bg-primary/20 text-primary-text`.
     - **Spoiler** (`spoiler`) — quando ativo, `border-warning bg-warning/15 text-warning`.

---

## 6. Critérios de Aceite (Gherkin)

### Cenário 1: Consistência de tokens sem cores soltas
- **Dado** qualquer componente novo criado nas features `movies` ou `reviews`
- **Quando** revisado no PR
- **Então** todas as classes de cor pertencem aos tokens semânticos de `globals.css` e o documento `logbuster-web/docs/identidade-visual.md` remove o aviso de *"Provisório"* referenciando esta `SPEC-UI-01`.

### Cenário 2: Responsividade mobile e navegação por teclado
- **Dado** uma tela aberta em largura mobile (`375px`) ou desktop (`1280px`)
- **Quando** o usuário navega com `Tab` pelos controles de busca, ficha do filme, modal e botões de voto
- **Então** não há quebra horizontal (`overflow-x`) e todos os focos são claramente visíveis com `--color-primary-text`.

## 7. Definição de Pronto (DoD)
- [ ] `globals.css` validado com todos os tokens acima.
- [ ] `logbuster-web/docs/identidade-visual.md` atualizado apontando para esta SPEC oficial.
- [ ] Card `LB-008` atualizado com link para esta SPEC.
