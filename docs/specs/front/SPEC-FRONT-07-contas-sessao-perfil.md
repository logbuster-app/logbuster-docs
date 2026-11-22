# SPEC-FRONT-07 — Contas, Sessão e Perfil Público

- **Status:** Draft
- **Card(s) Trello:** [Front 1](https://trello.com/b/qTbAU9jJ/equipe-alpha-gp) *(atualizar com o link do card)*
- **Repositório:** `logbuster-web`
- **Responsável:** Thyago Viana, com apoio de Marcelo Palmeira (Back A)
- **Dependências:** [`SPEC-BACK-08`](../back/SPEC-BACK-08-auth-perfil-publico.md), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md), [`SPEC-FRONT-03`](SPEC-FRONT-03-lista-ordenacao-interacoes-avaliacoes.md), [`SPEC-FRONT-04`](SPEC-FRONT-04-modal-cadastro-avaliacao.md), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md), [`CR-01`](../cr/CR-01-conta-obrigatoria-publicar.md) (back exige conta para publicar)
- **Entrega:** congelamento em 24/11/2026

---

## 1. Contexto e Objetivo

O back-end de contas e perfis já está na `main` ([`SPEC-BACK-08`](../back/SPEC-BACK-08-auth-perfil-publico.md)). Esta SPEC cobre o lado do front: cadastro, login, sessão, avatar no cabeçalho, publicação de avaliação com a conta, perfil público com histórico e edição do perfil pelo titular (bio, avatar, foto e fundo).

**Publicar avaliação exige conta** (decisão de 22/11/2026, registrada no [`CR-01`](../cr/CR-01-conta-obrigatoria-publicar.md) para o back). Navegar, buscar, ler avaliações e ver perfis continuam abertos a qualquer visitante. Os votos seguem anônimos nesta SPEC. O perfil também precisa receber, sem retrabalho, o **arquétipo** e o botão da **Máquina do Tempo**, que serão feitos por outra pessoa (Front 3) em cima desta tela. Por isso a tela expõe pontos de encaixe (seção 4.7).

> **Identidade visual:** seguir a [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md) (revisão 2, **"Cartaz do Cinema Novo"**: vermelho `primary`, osso `foreground`, cantos retos, títulos em `font-display`), em especial a seção 6, com os padrões de formulário, menu de conta e perfil. Em caso de divergência, valem os tokens de `src/app/globals.css`.

## 2. Escopo

- **Inclui:**
  - Telas `/register` (cadastro) e `/login`, e ação **Sair**.
  - Sessão no navegador (token JWT), restauração ao recarregar e tratamento de expiração.
  - Menu de conta no cabeçalho (Entrar e Criar conta / avatar, nome e Sair).
  - Avaliar passa a exigir login: botão "Avaliar filme" leva ao login quando não há sessão, e o modal perde o campo de nome.
  - Rota `/users/[id]` com perfil público e histórico paginado das avaliações da conta.
  - Edição do perfil pelo titular: bio, avatar de catálogo, foto de perfil e fundo.
  - Ajustes em `shared/lib/apiClient.ts` para corpo binário e resposta `204`.
- **Não inclui:** recorte ou edição de imagem no navegador, login social, recuperação de senha, confirmação de e-mail, seguir pessoas, mensagens, perfil privado, temas e links externos, migração de avaliações antigas para contas, arquétipo e Máquina do Tempo (entram pelos pontos de encaixe, em outro card).

## 3. Regras de Negócio

- **RN-01 (Avaliar exige conta).** Só publicar avaliação exige sessão. Buscar, ver a ficha, ler avaliações, ver perfis e votar continuam abertos (os votos seguem com `X-Visitor-Id`). Sem sessão, o botão **"Avaliar filme"** continua visível e leva para `/login?next=/movies/{id}`. Depois do login, a pessoa volta para o filme.
- **RN-02 (Sessão).** O token e os dados mínimos de quem está logado (`userId`, `displayName`) ficam em `localStorage` sob a chave `logbuster:session`, junto com o instante de expiração calculado a partir de `expires_in`. Não há refresh token: ao expirar (1 hora por padrão), a sessão é descartada e a pessoa volta ao estado anônimo. Se `localStorage` estiver indisponível, a sessão vale só até recarregar a página, sem quebrar a aplicação.
- **RN-03 (Token enviado automaticamente).** Toda chamada ao `backendApi` envia `Authorization: Bearer <token>` quando há sessão válida, além do `X-Visitor-Id`. Nenhum componente ou hook monta esse header.
- **RN-04 (Expiração e 401).** Uma resposta `401` a uma chamada que **enviou token** limpa a sessão e atualiza a interface para o estado anônimo, sem recarregar a página nem apagar o que a pessoa digitou. Um `401` de `/auth/login` (credenciais erradas) **não** limpa nada e mostra a mensagem do envelope no formulário.
- **RN-05 (Validação no cliente).** Reflete o back-end e nunca o substitui (os erros `422` e `409` sempre são exibidos):
  - Nome de exibição: obrigatório, 2 a 50 caracteres após `trim()`.
  - E-mail: obrigatório e com formato básico (`algo@algo.algo`).
  - Senha no cadastro: 8 a 128 caracteres. No login: obrigatória.
  - Bio: até 160 caracteres, com contador. Pode ser limpa.
- **RN-06 (Redirecionamento).** Após login ou cadastro, volta para a página de origem (`?next=`) ou para `/`. O valor de `next` só é aceito se começar com `/` e **não** começar com `//` (evita redirecionamento aberto). Após o cadastro, a pessoa é autenticada automaticamente (cadastro seguido de login com as mesmas credenciais).
- **RN-07 (Modal de avaliação).** O modal só abre com sessão. Ele não tem campo de nome, mostra "Publicando como *nome*" e não envia `author_name`. O nome salvo em `localStorage` (`logbuster:review_author_name`) deixa de ser usado. Se a publicação retornar `401` (sessão expirada, RN-04), o modal **preserva** nota, destaques, texto e spoiler e mostra um aviso com o botão "Entrar de novo", que leva ao login e volta ao filme. Guardar o rascunho para reabri-lo depois é desejável, mas não obrigatório.
- **RN-08 (Perfil público).** Qualquer visitante abre `/users/[id]`. Mostra fundo, foto ou avatar, nome, data de entrada ("Entrou em *mês de ano*"), bio quando existir e o histórico. E-mail nunca aparece. Perfil inexistente (`404`) usa a tela `not-found` da rota.
- **RN-09 (Imagem exibida).** Prioridade: `avatar_image_url` (foto) → ilustração do `avatar_id` → inicial do nome sobre `bg-surface-raised`. Fundo: `background_image_url` ou o fundo padrão do front (faixa chapada `bg-brand-800`). URLs relativas são resolvidas com `new URL(url, env.apiBaseUrl)`.
- **RN-10 (Edição só do titular).** O botão **Editar perfil** só aparece quando `viewer.userId` é igual ao `user_id` do perfil aberto. Os endpoints também validam isso, então a interface nunca é a única barreira.
- **RN-11 (Imagens).** Antes de enviar, validar tipo (JPEG, PNG ou WebP) e tamanho (até 8 MiB). Enviar o próprio `File` no corpo do `PUT`, com `Content-Type` do arquivo, sem `FormData` e sem Base64. Foto e fundo são independentes entre si e do `PATCH` de bio e avatar. Em erro, a imagem atual permanece e a mensagem do envelope aparece no editor. Remover usa `DELETE` e recarrega o perfil.
- **RN-12 (Histórico).** Lista paginada (`page_size` 10), da mais recente para a mais antiga, só com avaliações vinculadas à conta. Cada item reaproveita o card de avaliação existente (com ocultação de spoiler e votos), mostra o filme (título e link para `/movies/{movieId}`) e acompanha a regra de spoiler de [`SPEC-FRONT-03`](SPEC-FRONT-03-lista-ordenacao-interacoes-avaliacoes.md). Perfil sem avaliações mostra estado vazio e continua acessível.
- **RN-13 (Título do filme).** O item do histórico traz só `movie_id`. O título vem da ficha do filme pelo serviço que já existe em `features/movies`, com falha silenciosa: sem título, exibe o próprio `movie_id` como link.

## 4. Contrato Técnico

### 4.1 Estrutura de arquivos esperada

Respeita as regras de [`docs/arquitetura.md`](https://github.com/logbuster-app/logbuster-web/blob/main/docs/arquitetura.md): sem `fetch` em componente ou hook, services por feature, features não se importam entre si, `shared` não importa de `features`, modelo de domínio em camelCase via mapper, testes ao lado do código.

```text
src/
  app/
    login/page.tsx                 Fina: monta <LoginForm />
    register/page.tsx              Fina: monta <RegisterForm />
    users/[id]/page.tsx            Compõe profile + reviews + movies (única que cruza features)
    users/[id]/not-found.tsx
    layout.tsx                     Envolve o app em <SessionProvider />
  features/
    auth/
      api/auth.ts                  register, login, fetchCurrentUser
      components/LoginForm.tsx
      components/RegisterForm.tsx
      validation.ts                Regras da RN-05
      index.ts
    profile/
      api/profile.ts               fetchProfile, updateProfile, uploadProfileImage, deleteProfileImage, fetchAvatars
      components/ProfileHeader.tsx Fundo, foto/avatar, nome, data, bio, slots
      components/Avatar.tsx        Prioridade da RN-09
      components/ProfileEditModal.tsx
      components/AvatarPicker.tsx
      components/ImageField.tsx    Enviar, trocar e remover (foto e fundo)
      types.ts, validation.ts, index.ts
    reviews/                       (existente, ajustes na seção 4.5)
    movies/                        (existente, novo export <MovieTitleLink />)
  shared/
    lib/session.ts                 Leitura e gravação do localStorage, getAccessToken()
    contexts/SessionContext.tsx    SessionProvider e useSession()
    components/AccountMenu.tsx     Menu de conta do cabeçalho
public/avatars/avatar-01.svg … avatar-08.svg
```

### 4.2 Sessão (`shared`)

- `useSession()` devolve `{ viewer: { userId, displayName } | null, signIn(input), signOut() }`.
- `signIn({ accessToken, expiresIn, viewer })` grava a sessão (RN-02) e atualiza o contexto. Quem chama é a feature `auth`, depois de `login` e `fetchCurrentUser`.
- `getAccessToken()` retorna o token válido ou `null`, descartando token vencido.
- `shared/api/clients.ts`: o `backendApi` passa a incluir `Authorization` (RN-03) e um `onError` que, em `401` com token enviado, limpa a sessão (RN-04). O `SessionProvider` reage a essa limpeza (evento interno) e também à alteração da chave em outra aba (`storage`).
- Atenção ao SSR: no servidor não há sessão e o header de autorização não é enviado, como já acontece com `X-Visitor-Id`.

### 4.3 Ajustes no `apiClient` (`shared/lib/apiClient.ts`)

**Nenhuma mudança no back.** Para enviar arquivos, o cliente HTTP precisa de dois ajustes:

1. **Corpo binário.** Aceitar `body` do tipo `Blob`/`File`, enviado como está, com o `Content-Type` informado por quem chama, sem `JSON.stringify`. O envio de objetos JSON continua igual.
2. **Resposta `204`.** Não chamar `res.json()` quando o status for `204` ou o corpo vier vazio (o `DELETE` das imagens retorna `204`).
3. Cobrir os dois casos com testes unitários.

### 4.4 Endpoints consumidos ([`SPEC-BACK-08`](../back/SPEC-BACK-08-auth-perfil-publico.md))

| Uso | Chamada |
| :--- | :--- |
| Cadastro | `POST /api/v1/auth/register` com `{display_name, email, password}` |
| Login | `POST /api/v1/auth/login` com `{email, password}` → `{access_token, expires_in}` |
| Sessão | `GET /api/v1/auth/me` (Bearer) |
| Perfil | `GET /api/v1/users/{user_id}` |
| Histórico | `GET /api/v1/users/{user_id}/reviews?page=&page_size=` |
| Catálogo de avatares | `GET /api/v1/avatars` |
| Editar bio e avatar | `PATCH /api/v1/users/me` com `{bio?, avatar_id?}` |
| Enviar foto ou fundo | `PUT /api/v1/users/me/images/{avatar\|background}` com o arquivo no corpo → `{url}` |
| Remover foto ou fundo | `DELETE /api/v1/users/me/images/{avatar\|background}` (`204`) |
| Publicar avaliação | `POST /api/v1/reviews` com Bearer obrigatório (CR-01) |

Modelo de domínio do perfil (camelCase, via mapper): `userId`, `displayName`, `createdAt`, `bio`, `avatarId`, `avatarImageUrl` e `backgroundImageUrl` (absolutas), `archetype` (`{ title, description } | null`, repassado como veio para o encaixe da seção 4.7).

### 4.5 Ajustes em `features/reviews`

- `ReviewModal` e `validateReview`: remover o campo e a validação de `authorName` (RN-07). O botão que abre o modal fica na ficha do filme e, sem sessão, navega para o login em vez de abrir o modal (RN-01). A ficha lê a sessão por `useSession()` de `shared`.
- `createReview`: deixa de enviar `author_name`. O restante do payload e do tratamento de erro permanece, e o `401` é tratado como na RN-07.
- Novo service `fetchUserReviews(userId, page)` reaproveitando o mapper `toReview` (os itens têm o mesmo formato da lista de avaliações do filme).
- Novo componente `UserReviewHistory` (paginação "Carregar mais" ou páginas, a critério da implementação), recebendo `renderMovie: (movieId: string) => ReactNode` para não importar `features/movies` (a página `users/[id]` fornece `<MovieTitleLink />`).
- `ReviewCard` ganha uma prop opcional para exibir o filme acima do texto, sem alterar o uso atual na ficha do filme.

### 4.6 Ajustes em `features/movies`

- Novo export `MovieTitleLink({ movieId })`: link para `/movies/{movieId}` com o título do filme (RN-13), reaproveitando o service e o hook de detalhe existentes.

### 4.7 Pontos de encaixe para o Front 3 (arquétipo e Máquina do Tempo)

`ProfileHeader` recebe duas props opcionais, `archetypeSlot?: ReactNode` e `actionsSlot?: ReactNode`. A página `users/[id]` as preenche com o que existir. Esta SPEC **não** implementa nenhum dos dois. Sem os slots preenchidos, a tela fica completa e sem espaço vazio. O objeto `archetype` já chega no modelo de domínio (seção 4.4) para quem for usá-lo.

### 4.8 Visual (identidade "Cartaz do Cinema Novo")

Usar somente tokens e padrões que já existem no front. Nada de hex solto e nada de cantos arredondados (`rounded-card` é `0`).

| Elemento | Tratamento |
| :--- | :--- |
| Títulos de página e de seção | `font-display text-foreground` |
| Formulários (login, cadastro, editor) | Mesmo padrão do `ReviewModal`: rótulo `text-xs font-bold uppercase tracking-wide`, campo `h-11 border-2 border-border bg-background`, foco `border-foreground`, obrigatório com `text-danger` |
| Botão primário | `min-h-11 bg-primary text-primary-foreground hover:bg-primary-hover font-bold uppercase tracking-wide` |
| Botão secundário | `border-2 border-border text-muted hover:border-foreground hover:text-foreground` |
| Erros | `role="alert"` com `border-2 border-danger text-danger` |
| Banner do perfil | `aspect-[16/9]`, altura máxima moderada, `object-cover`. Padrão `bg-brand-800` |
| Avatar | Quadrado `size-24 sm:size-32 border-2 border-foreground`, sobreposto à base do banner |
| Menu de conta no cabeçalho | Sobre a faixa `bg-primary`: texto `text-primary-foreground`, foco `focus-visible:outline-primary-foreground` (igual ao logotipo) |
| Estados vazios e erros de carga | `StateMessage` existente |

Acessibilidade: campos com `label` associado, `autocomplete` correto (`email`, `current-password`, `new-password`, `nickname`), erros anunciados, navegação completa por teclado, alvos de toque de no mínimo 44 px, sem rolagem horizontal em 375 px.

### 4.9 Avatares de catálogo

O back devolve os IDs `avatar-01` a `avatar-08`. O front fornece as ilustrações em `public/avatars/`. O Thyago cria 8 ilustrações geométricas simples em SVG, nas cores dos tokens e com cantos retos, distintas entre si à primeira vista. Cada uma tem um rótulo acessível (por exemplo, "Avatar 1 — claquete"). Se alguma não ficar pronta no congelamento, o fallback da RN-09 (inicial do nome) cobre a tela.

### 4.10 Dados de demonstração

Preparar ao menos uma conta de demonstração com foto, fundo, bio e 3 ou mais avaliações, para o perfil aparecer completo no pitch (e para o arquétipo do Front 3 aparecer). Usar imagens próprias da equipe ou com licença livre.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Cadastro e login
- **Dado** uma pessoa sem conta em `/register`
- **Quando** preenche nome, e-mail novo e senha válida e envia
- **Então** a conta é criada, a pessoa é autenticada e vai para a página de origem, com o avatar e o nome no cabeçalho
- **E** um e-mail já cadastrado (`409`) mostra a mensagem do servidor no formulário, mantendo os campos preenchidos, exceto a senha.

### Cenário 2: Credenciais inválidas não derrubam nada
- **Dado** a tela `/login`
- **Quando** a senha está errada (`401`)
- **Então** a mensagem de erro aparece no formulário e o cabeçalho continua no estado anônimo.

### Cenário 3: Sessão sobrevive ao recarregar e expira
- **Dado** uma pessoa logada
- **Quando** recarrega a página
- **Então** continua logada
- **E** com token vencido, ou com `401` em qualquer chamada autenticada, a interface volta ao estado anônimo sem recarregar a página.

### Cenário 4: Avaliar exige login
- **Dado** a ficha de um filme
- **Quando** uma pessoa logada abre o modal
- **Então** vê "Publicando como *nome*", não vê campo de nome, e a avaliação publicada aparece vinculada à conta no perfil
- **E** sem sessão, clicar em "Avaliar filme" leva ao login e, depois de entrar, volta para o mesmo filme.

### Cenário 5: Sessão expira durante o preenchimento
- **Dado** o modal aberto, logado, com texto digitado
- **Quando** a publicação retorna `401`
- **Então** o texto e as seleções permanecem e aparece o aviso com o botão "Entrar de novo".

### Cenário 6: Perfil público e histórico
- **Dado** um usuário com avaliações em datas distintas
- **Quando** um visitante abre `/users/{id}`
- **Então** vê nome, data de entrada, bio, imagens e as avaliações da conta, da mais recente para a mais antiga, cada uma com o título do filme e link para a ficha
- **E** um perfil sem avaliações mostra o estado vazio, e um `id` inexistente mostra a página não encontrada.

### Cenário 6b: Spoiler oculto no perfil
- **Dado** um perfil cujo histórico tem uma avaliação com `is_spoiler = true` (marcada pelo autor ou pela IA, conforme o [`CR-02`](../cr/CR-02-spoiler-ia-pos-publicacao.md)) e outra com 10 ou mais votos de spoiler (limiar do CR-02)
- **Quando** um visitante abre o perfil
- **Então** as duas aparecem com o texto oculto e o aviso "Esta avaliação contém spoiler", com o botão "Mostrar análise", igual à ficha do filme
- **E** o título do filme e a nota continuam visíveis.

### Cenário 7: Editar perfil
- **Dado** o titular autenticado em seu próprio perfil
- **Quando** altera bio e avatar e salva
- **Então** o perfil exibe os novos valores
- **E** em outro perfil, ou sem sessão, o botão **Editar perfil** não existe.

### Cenário 8: Foto e fundo
- **Dado** o editor aberto pelo titular
- **Quando** envia uma foto válida e um fundo válido
- **Então** a foto aparece no perfil e o fundo no banner, também para quem visita o perfil
- **E** remover a foto volta ao avatar escolhido, remover o fundo volta ao padrão, e um arquivo recusado (tipo errado ou acima de 8 MiB) mantém a imagem anterior com a mensagem de erro visível.

### Cenário 9: Encaixes livres
- **Dado** o perfil sem nada nos slots
- **Quando** aberto
- **Então** não há lacuna visual
- **E** ao passar `archetypeSlot` e `actionsSlot`, os conteúdos aparecem no cabeçalho do perfil.

## 6. Definição de Pronto (DoD)

- [ ] Telas `/login`, `/register` e `/users/[id]` funcionando contra a API local e contra o deploy de teste (23/11).
- [ ] Sessão restaurada ao recarregar, expiração tratada e `Authorization` enviado só pelo `apiClient`, sem `fetch` em componente ou hook.
- [ ] 8 avatares em SVG com seletor no editor.
- [ ] Conta de demonstração com foto, fundo, bio e avaliações.
- [ ] `apiClient` com corpo binário e `204`, com testes.
- [ ] Avaliar exigindo login, com modal sem campo de nome e tratamento de sessão expirada.
- [ ] Slots `archetypeSlot` e `actionsSlot` entregues e documentados para o Front 3.
- [ ] Testes ao lado do código e `npm run check` e `npm run build` passando no CI.
- [ ] Documentação criada em `logbuster-web/docs/features/contas-e-perfil.md` e `docs/arquitetura.md` atualizado (sessão em `shared`).
- [ ] SPEC e card atualizados com os links, e status da SPEC alterado de `Draft` para `Approved` após revisão.
