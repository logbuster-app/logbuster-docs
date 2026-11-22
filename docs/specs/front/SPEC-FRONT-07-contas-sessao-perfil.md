# SPEC-FRONT-07 — Contas e Sessão

- **Status:** Draft
- **Card(s) Trello:** `[FRONT-07]` *(atualizar com o link do card)*
- **Repositório:** `logbuster-web`
- **Responsável:** Thyago Viana, com apoio de Marcelo Palmeira (Back A)
- **Dependências:** [`SPEC-BACK-08`](../back/SPEC-BACK-08-auth-perfil-publico.md), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md), [`SPEC-FRONT-04`](SPEC-FRONT-04-modal-cadastro-avaliacao.md), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md), [`CR-01`](../cr/CR-01-conta-obrigatoria-publicar.md) (back exige conta para publicar)
- **Entrega:** congelamento em 24/11/2026

> **Divisão de 22/11/2026:** a página de perfil público (histórico, edição, imagens e avatares) saiu desta SPEC e foi para a [`SPEC-FRONT-09`](SPEC-FRONT-09-perfil-publico-arquetipo.md) (Davi), junto com o arquétipo. Esta SPEC fica só com contas, sessão e publicação com conta.

---

## 1. Contexto e Objetivo

O back-end de contas já está na `main` ([`SPEC-BACK-08`](../back/SPEC-BACK-08-auth-perfil-publico.md)) e já exige conta para publicar ([`CR-01`](../cr/CR-01-conta-obrigatoria-publicar.md)). Esta SPEC cobre o lado do front: cadastro, login, sessão, menu de conta no cabeçalho e o modal de avaliação usando a conta.

**Publicar avaliação exige conta.** Navegar, buscar, ler avaliações, ver perfis e votar continuam abertos a qualquer visitante (os votos seguem com `X-Visitor-Id`).

> **Identidade visual:** seguir a [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md) (revisão 2, **"Cartaz do Cinema Novo"**), em especial a seção 6, com os padrões de formulário e de menu de conta. Em caso de divergência, valem os tokens de `src/app/globals.css`.

> **Atenção:** o back já exige conta desde 22/11. Até este card entrar, **o front em produção não consegue publicar avaliações**. Este é o card mais urgente da Parte 2.

## 2. Escopo

- **Inclui:**
  - Telas `/register` (cadastro) e `/login`, e ação **Sair**.
  - Sessão no navegador (token JWT), restauração ao recarregar e tratamento de expiração.
  - Envio automático do token em todas as chamadas ao back.
  - Menu de conta no cabeçalho (Entrar e Criar conta, ou inicial, nome e Sair, com link para o próprio perfil).
  - Avaliar exige login: "Avaliar filme" leva ao login quando não há sessão, e o modal perde o campo de nome.
- **Não inclui:** a página de perfil público e a edição de perfil ([`SPEC-FRONT-09`](SPEC-FRONT-09-perfil-publico-arquetipo.md)), login social, recuperação de senha, confirmação de e-mail e votos com conta (CR-03).

## 3. Regras de Negócio

- **RN-01 (Avaliar exige conta).** Só publicar avaliação exige sessão. Sem sessão, o botão **"Avaliar filme"** continua visível e leva para `/login?next=/movies/{id}`. Depois do login, a pessoa volta para o filme.
- **RN-02 (Sessão).** O token e os dados mínimos de quem está logado (`userId`, `displayName`) ficam em `localStorage` sob a chave `logbuster:session`, junto com o instante de expiração calculado a partir de `expires_in`. Não há refresh token: ao expirar (1 hora por padrão), a sessão é descartada e a pessoa volta ao estado deslogado. Se `localStorage` estiver indisponível, a sessão vale só até recarregar a página, sem quebrar a aplicação.
- **RN-03 (Token enviado automaticamente).** Toda chamada ao `backendApi` envia `Authorization: Bearer <token>` quando há sessão válida, além do `X-Visitor-Id`. Nenhum componente ou hook monta esse header.
- **RN-04 (Expiração e 401).** Uma resposta `401` a uma chamada que **enviou token** limpa a sessão e atualiza a interface para o estado deslogado, sem recarregar a página nem apagar o que a pessoa digitou. Um `401` de `/auth/login` (credenciais erradas) **não** limpa nada e mostra a mensagem do envelope no formulário.
- **RN-05 (Validação no cliente).** Reflete o back-end e nunca o substitui (os erros `422` e `409` sempre são exibidos):
  - Nome de exibição: obrigatório, 2 a 50 caracteres após `trim()`.
  - E-mail: obrigatório e com formato básico (`algo@algo.algo`).
  - Senha no cadastro: 8 a 128 caracteres. No login: obrigatória.
- **RN-06 (Redirecionamento).** Após login ou cadastro, volta para a página de origem (`?next=`) ou para `/`. O valor de `next` só é aceito se começar com `/` e **não** começar com `//` (evita redirecionamento aberto). Após o cadastro, a pessoa é autenticada automaticamente (cadastro seguido de login com as mesmas credenciais).
- **RN-07 (Modal de avaliação).** O modal só abre com sessão. Ele não tem campo de nome, mostra "Publicando como *nome*" e não envia `author_name`. O nome salvo em `localStorage` (`logbuster:review_author_name`) deixa de ser usado. Se a publicação retornar `401` (sessão expirada, RN-04), o modal **preserva** nota, destaques, texto e spoiler e mostra um aviso com o botão "Entrar de novo", que leva ao login e volta ao filme.
- **RN-08 (Menu de conta).** Deslogado: links "Entrar" e "Criar conta". Logado: inicial do nome, nome de exibição, link para `/users/{userId}` (página da [`SPEC-FRONT-09`](SPEC-FRONT-09-perfil-publico-arquetipo.md)) e "Sair". O menu não busca foto nem avatar, para não depender da feature de perfil.

## 4. Contrato Técnico

### 4.1 Estrutura de arquivos esperada

Respeita [`docs/arquitetura.md`](https://github.com/logbuster-app/logbuster-web/blob/main/docs/arquitetura.md): sem `fetch` em componente ou hook, services por feature, features não se importam entre si, `shared` não importa de `features`, modelo de domínio em camelCase via mapper, testes ao lado do código.

```text
src/
  app/
    login/page.tsx                 Fina: monta <LoginForm />
    register/page.tsx              Fina: monta <RegisterForm />
    layout.tsx                     Envolve o app em <SessionProvider />
  features/
    auth/
      api/auth.ts                  register, login, fetchCurrentUser
      components/LoginForm.tsx
      components/RegisterForm.tsx
      validation.ts                Regras da RN-05
      index.ts
    reviews/                       (existente, ajustes na seção 4.4)
  shared/
    lib/session.ts                 Leitura e gravação do localStorage, getAccessToken()
    contexts/SessionContext.tsx    SessionProvider e useSession()
    components/AccountMenu.tsx     Menu de conta do cabeçalho (RN-08)
```

### 4.2 Sessão (`shared`)

- `useSession()` devolve `{ viewer: { userId, displayName } | null, signIn(input), signOut() }`. **A SPEC-FRONT-09 usa este hook** para decidir se mostra "Editar perfil".
- `signIn({ accessToken, expiresIn, viewer })` grava a sessão (RN-02) e atualiza o contexto. Quem chama é a feature `auth`, depois de `login` e `fetchCurrentUser`.
- `getAccessToken()` retorna o token válido ou `null`, descartando token vencido.
- `shared/api/clients.ts`: o `backendApi` passa a incluir `Authorization` (RN-03) e um `onError` que, em `401` com token enviado, limpa a sessão (RN-04). O `SessionProvider` reage a essa limpeza (evento interno) e também à alteração da chave em outra aba (`storage`).
- Atenção ao SSR: no servidor não há sessão e o header de autorização não é enviado, como já acontece com `X-Visitor-Id`.

### 4.3 Endpoints consumidos ([`SPEC-BACK-08`](../back/SPEC-BACK-08-auth-perfil-publico.md))

| Uso | Chamada |
| :--- | :--- |
| Cadastro | `POST /api/v1/auth/register` com `{display_name, email, password}` |
| Login | `POST /api/v1/auth/login` com `{email, password}` → `{access_token, expires_in}` |
| Sessão | `GET /api/v1/auth/me` (Bearer) |
| Publicar avaliação | `POST /api/v1/reviews` com Bearer obrigatório (CR-01) |

### 4.4 Ajustes em `features/reviews`

- `ReviewModal` e `validateReview`: remover o campo e a validação de `authorName` (RN-07). O botão que abre o modal fica na ficha do filme e, sem sessão, navega para o login em vez de abrir o modal (RN-01). A ficha lê a sessão por `useSession()` de `shared`.
- `createReview`: deixa de enviar `author_name`. O restante do payload e do tratamento de erro permanece, e o `401` é tratado como na RN-07.

### 4.5 Visual (identidade "Cartaz do Cinema Novo")

| Elemento | Tratamento |
| :--- | :--- |
| Títulos | `font-display text-foreground` |
| Formulários (login e cadastro) | Mesmo padrão do `ReviewModal`: rótulo `text-xs font-bold uppercase tracking-wide`, campo `h-11 border-2 border-border bg-background`, foco `border-foreground`, obrigatório com `text-danger` |
| Botão primário | `min-h-11 bg-primary text-primary-foreground hover:bg-primary-hover font-bold uppercase tracking-wide` |
| Botão secundário | `border-2 border-border text-muted hover:border-foreground hover:text-foreground` |
| Erros | `role="alert"` com `border-2 border-danger text-danger` |
| Menu de conta | Sobre a faixa `bg-primary`: texto `text-primary-foreground`, foco `focus-visible:outline-primary-foreground` (igual ao logotipo). Inicial em quadrado `size-9 border-2 border-primary-foreground` |

Acessibilidade: campos com `label` associado, `autocomplete` correto (`email`, `current-password`, `new-password`, `nickname`), erros anunciados, navegação completa por teclado, alvos de toque de no mínimo 44 px, sem rolagem horizontal em 375 px.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Cadastro e login
- **Dado** uma pessoa sem conta em `/register`
- **Quando** preenche nome, e-mail novo e senha válida e envia
- **Então** a conta é criada, a pessoa é autenticada e vai para a página de origem, com a inicial e o nome no cabeçalho
- **E** um e-mail já cadastrado (`409`) mostra a mensagem do servidor no formulário, mantendo os campos preenchidos, exceto a senha.

### Cenário 2: Credenciais inválidas não derrubam nada
- **Dado** a tela `/login`
- **Quando** a senha está errada (`401`)
- **Então** a mensagem de erro aparece no formulário e o cabeçalho continua deslogado.

### Cenário 3: Sessão sobrevive ao recarregar e expira
- **Dado** uma pessoa logada
- **Quando** recarrega a página
- **Então** continua logada
- **E** com token vencido, ou com `401` em qualquer chamada autenticada, a interface volta ao estado deslogado sem recarregar a página.

### Cenário 4: Avaliar exige login
- **Dado** a ficha de um filme
- **Quando** uma pessoa logada abre o modal
- **Então** vê "Publicando como *nome*", não vê campo de nome, e a avaliação é publicada vinculada à conta
- **E** sem sessão, clicar em "Avaliar filme" leva ao login e, depois de entrar, volta para o mesmo filme.

### Cenário 5: Sessão expira durante o preenchimento
- **Dado** o modal aberto, logado, com texto digitado
- **Quando** a publicação retorna `401`
- **Então** o texto e as seleções permanecem e aparece o aviso com o botão "Entrar de novo".

### Cenário 6: Sair
- **Dado** uma pessoa logada
- **Quando** clica em "Sair"
- **Então** a sessão é apagada, o cabeçalho volta a mostrar "Entrar" e "Criar conta", e "Avaliar filme" volta a levar ao login.

## 6. Definição de Pronto (DoD)

- [ ] Telas `/login` e `/register` funcionando contra a API local e contra o deploy de teste (23/11).
- [ ] Sessão restaurada ao recarregar, expiração tratada e `Authorization` enviado só pelo `apiClient`, sem `fetch` em componente ou hook.
- [ ] `useSession()` exportado de `shared` e documentado para a SPEC-FRONT-09.
- [ ] Avaliar exigindo login, com modal sem campo de nome e tratamento de sessão expirada.
- [ ] Testes ao lado do código e `npm run check` e `npm run build` passando no CI.
- [ ] Documentação criada em `logbuster-web/docs/features/contas-e-sessao.md` e `docs/arquitetura.md` atualizado (sessão em `shared`).
- [ ] Status alterado de `Draft` para `Approved` após revisão.
