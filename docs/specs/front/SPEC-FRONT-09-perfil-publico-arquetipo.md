# SPEC-FRONT-09 — Perfil Público e Arquétipo do Cinéfilo

- **Status:** Draft
- **Card(s) Trello:** `[FRONT-09]` *(atualizar com o link do card)*
- **Repositório:** `logbuster-web`
- **Responsável:** Davi, com apoio de Marcelo Palmeira (Back A) e João Victor (Back B)
- **Dependências:** [`SPEC-BACK-08`](../back/SPEC-BACK-08-auth-perfil-publico.md) (perfil e imagens), [`SPEC-BACK-09`](../back/SPEC-BACK-09-ia-e-arquetipo.md) (arquétipo), [`SPEC-FRONT-03`](SPEC-FRONT-03-lista-ordenacao-interacoes-avaliacoes.md) (card de avaliação), [`SPEC-FRONT-07`](SPEC-FRONT-07-contas-sessao-perfil.md) (sessão, `useSession()`), [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md), [`CR-02`](../cr/CR-02-spoiler-ia-pos-publicacao.md) (limiar de 10 votos)
- **Entrega:** congelamento em 24/11/2026

> **Origem:** esta SPEC reúne a parte de perfil que saiu da SPEC-FRONT-07 na divisão de 22/11/2026 e o arquétipo, que antes seria encaixado por espaços vazios. Agora a mesma pessoa constrói a página inteira.

---

## 1. Contexto e Objetivo

O back já entrega o perfil público, o histórico de avaliações, a edição de bio e avatar, o envio de foto e fundo ([`SPEC-BACK-08`](../back/SPEC-BACK-08-auth-perfil-publico.md)) e o **arquétipo do cinéfilo** dentro da resposta do perfil ([`SPEC-BACK-09`](../back/SPEC-BACK-09-ia-e-arquetipo.md)). Esta SPEC cria a página `/users/[id]` no front com tudo isso.

**Ordem de entrega sugerida:** primeiro a parte pública (perfil, histórico e arquétipo), que não depende de login e pode começar já. Depois a edição, que depende de `useSession()` da SPEC-FRONT-07 (Thyago).

> **Identidade visual:** seguir a [`SPEC-UI-01`](../transversais/SPEC-UI-01-identidade-visual.md), seção 6, itens 4 (perfil) e 7 (arquétipo). Em caso de divergência, valem os tokens de `src/app/globals.css`.

## 2. Escopo

- **Inclui:**
  - Rota `/users/[id]` com perfil público: fundo, foto ou avatar, nome, data de entrada, bio e **arquétipo**.
  - Histórico paginado das avaliações da conta, com o título do filme.
  - Edição pelo titular: bio, avatar de catálogo (8 opções), foto de perfil e fundo.
  - 8 ilustrações simples de avatar em SVG.
  - Ajustes em `shared/lib/apiClient.ts` para corpo binário e resposta `204`.
- **Não inclui:** login e sessão (SPEC-FRONT-07), Máquina do Tempo ([`SPEC-FRONT-10`](SPEC-FRONT-10-maquina-do-tempo.md), condicional, que depois acrescenta um botão no `ProfileHeader` e reaproveita `fetchUserReviews` e `MovieTitleLink` desta SPEC), recorte de imagem, seguir pessoas, perfil privado, temas e links externos.

## 3. Regras de Negócio

### Perfil e histórico
- **RN-01 (Perfil público).** Qualquer visitante abre `/users/[id]`. Mostra fundo, foto ou avatar, nome, data de entrada ("Entrou em *mês de ano*"), bio quando existir, arquétipo e histórico. E-mail nunca aparece. Perfil inexistente (`404`) usa a tela `not-found` da rota.
- **RN-02 (Imagem exibida).** Prioridade: `avatar_image_url` (foto) → ilustração do `avatar_id` → inicial do nome sobre `bg-surface-raised`. Fundo: `background_image_url` ou o fundo padrão (faixa chapada `bg-brand-800`). URLs relativas são resolvidas com `new URL(url, env.apiBaseUrl)`.
- **RN-03 (Histórico).** Lista paginada (`page_size` 10), da mais recente para a mais antiga, só com avaliações vinculadas à conta. Cada item reaproveita o `ReviewCard` existente (ocultação de spoiler e votos) e mostra o filme (título e link para `/movies/{movieId}`). Perfil sem avaliações mostra estado vazio e continua acessível.
- **RN-04 (Spoiler no perfil).** Avaliações com `is_spoiler = true` (autor ou IA) ou com 10 ou mais votos de spoiler aparecem ocultas, com "Mostrar análise", igual à ficha do filme. Título do filme e nota continuam visíveis. Isso vem do próprio `ReviewCard`, sem regra nova.
- **RN-05 (Título do filme).** O item do histórico traz só `movie_id`. O título vem da ficha do filme pelo serviço que já existe em `features/movies`, com falha silenciosa: sem título, exibe o próprio `movie_id` como link.

### Arquétipo
- **RN-06 (Arquétipo).** O campo `archetype` do perfil traz `{ title, description }` quando a conta tem pelo menos 3 avaliações vinculadas, e `null` abaixo disso. O back recalcula a cada consulta do perfil.
  - Com arquétipo: bloco com o título (por exemplo, "Crítico exigente") e a explicação curta.
  - Sem arquétipo: texto discreto "O arquétipo aparece a partir de 3 avaliações."
- **RN-07 (Arquétipo é público).** O arquétipo aparece para qualquer visitante, não só para o titular.

### Edição
- **RN-08 (Edição só do titular).** O botão **Editar perfil** só aparece quando `useSession().viewer.userId` é igual ao `user_id` do perfil aberto. Os endpoints também validam isso, então a interface nunca é a única barreira.
- **RN-09 (Bio e avatar).** Bio de até 160 caracteres, com contador, podendo ser limpa. Avatar escolhido entre os 8 do catálogo. Ambos salvos pelo `PATCH /users/me`.
- **RN-10 (Imagens).** Antes de enviar, validar tipo (JPEG, PNG ou WebP) e tamanho (até 8 MiB). Enviar o próprio `File` no corpo do `PUT`, com `Content-Type` do arquivo, sem `FormData` e sem Base64. Foto e fundo são independentes entre si e do `PATCH`. Em erro, a imagem atual permanece e a mensagem do envelope aparece no editor. Remover usa `DELETE` e recarrega o perfil.
- **RN-11 (Arquétipo após editar).** Depois de salvar, o perfil é recarregado, o que também atualiza o arquétipo.

## 4. Contrato Técnico

### 4.1 Estrutura de arquivos esperada

Respeita [`docs/arquitetura.md`](https://github.com/logbuster-app/logbuster-web/blob/main/docs/arquitetura.md): sem `fetch` em componente ou hook, services por feature, features não se importam entre si, modelo de domínio em camelCase via mapper, testes ao lado do código.

```text
src/
  app/
    users/[id]/page.tsx            Compõe profile + reviews + movies (única que cruza features)
    users/[id]/not-found.tsx
  features/
    profile/
      api/profile.ts               fetchProfile, updateProfile, uploadProfileImage, deleteProfileImage, fetchAvatars
      components/ProfileHeader.tsx Fundo, foto/avatar, nome, data, bio e arquétipo
      components/ArchetypeBadge.tsx Bloco do arquétipo (RN-06)
      components/Avatar.tsx        Prioridade da RN-02
      components/ProfileEditModal.tsx
      components/AvatarPicker.tsx
      components/ImageField.tsx    Enviar, trocar e remover (foto e fundo)
      types.ts, validation.ts, index.ts
    reviews/                       (existente, ajustes na seção 4.5)
    movies/                        (existente, novo export <MovieTitleLink />)
  shared/lib/apiClient.ts          corpo binário e 204 (seção 4.3)
public/avatars/avatar-01.svg … avatar-08.svg
```

### 4.2 Endpoints consumidos

| Uso | Chamada |
| :--- | :--- |
| Perfil (inclui arquétipo) | `GET /api/v1/users/{user_id}` |
| Histórico | `GET /api/v1/users/{user_id}/reviews?page=&page_size=` |
| Catálogo de avatares | `GET /api/v1/avatars` |
| Editar bio e avatar | `PATCH /api/v1/users/me` com `{bio?, avatar_id?}` (Bearer) |
| Enviar foto ou fundo | `PUT /api/v1/users/me/images/{avatar\|background}` com o arquivo no corpo → `{url}` (Bearer) |
| Remover foto ou fundo | `DELETE /api/v1/users/me/images/{avatar\|background}` → `204` (Bearer) |

O token é enviado automaticamente pelo `backendApi` (SPEC-FRONT-07). Este card não monta header de autorização.

Modelo de domínio do perfil (camelCase, via mapper): `userId`, `displayName`, `createdAt`, `bio`, `avatarId`, `avatarImageUrl` e `backgroundImageUrl` (absolutas), `archetype` (`{ title, description } | null`).

Exemplo de arquétipo na resposta:

```json
"archetype": {
  "title": "Crítico exigente",
  "description": "Costuma reservar as notas mais altas para poucos filmes."
}
```

### 4.3 Ajustes no `apiClient` (`shared/lib/apiClient.ts`)

1. **Corpo binário.** Aceitar `body` do tipo `Blob`/`File`, enviado como está, com o `Content-Type` informado por quem chama, sem `JSON.stringify`. O envio de objetos JSON continua igual.
2. **Resposta `204`.** Não chamar `res.json()` quando o status for `204` ou o corpo vier vazio.
3. Cobrir os dois casos com testes unitários.

### 4.4 Dependência da SPEC-FRONT-07

- Usa `useSession()` de `shared` só para a RN-08. Enquanto a SPEC-FRONT-07 não estiver na `main`, a parte pública funciona normalmente e o botão "Editar perfil" simplesmente não aparece.
- O menu de conta do cabeçalho (Thyago) já aponta para `/users/{userId}`.

### 4.5 Ajustes em `features/reviews`

- Novo service `fetchUserReviews(userId, page)` reaproveitando o mapper `toReview` (os itens têm o mesmo formato da lista de avaliações do filme).
- Novo componente `UserReviewHistory` (paginação "Carregar mais" ou páginas), recebendo `renderMovie: (movieId: string) => ReactNode` para não importar `features/movies`. A página `users/[id]` fornece `<MovieTitleLink />`.
- `ReviewCard` ganha uma prop opcional para exibir o filme acima do texto, sem alterar o uso atual na ficha do filme.

### 4.6 Ajustes em `features/movies`

- Novo export `MovieTitleLink({ movieId })`: link para `/movies/{movieId}` com o título do filme (RN-05), reaproveitando o service e o hook de detalhe existentes.

### 4.7 Avatares de catálogo

O back aceita os IDs `avatar-01` a `avatar-08`. O front fornece as ilustrações em `public/avatars/`: 8 ilustrações geométricas simples em SVG, nas cores dos tokens e com cantos retos, distintas entre si à primeira vista. Cada uma tem um rótulo acessível (por exemplo, "Avatar 1 — claquete"). Se alguma não ficar pronta no congelamento, o fallback da RN-02 (inicial do nome) cobre a tela.

### 4.8 Visual (identidade "Cartaz do Cinema Novo")

| Elemento | Tratamento |
| :--- | :--- |
| Banner | `aspect-[16/9]`, altura máxima moderada, `object-cover`. Padrão `bg-brand-800` |
| Avatar | Quadrado `size-24 sm:size-32 border-2 border-foreground`, sobreposto à base do banner |
| Nome | `font-display`; data de entrada em `text-muted`; bio em `font-reading` |
| Arquétipo | Bloco osso na linguagem da Nota LogBuster: título em `font-display` sobre `bg-foreground text-background` e explicação em `text-sm`. Sem arquétipo: `text-sm text-muted` |
| Editor | Modal no padrão do `ReviewModal` (`useModalBehavior`), formulários da SPEC-UI-01, seção 6, item 1 |
| Estados vazios e erros | `StateMessage` existente |

Acessibilidade: imagens com `alt` (foto: "Foto de *nome*"; fundo decorativo: `alt=""`), seletor de avatar navegável por teclado, alvos de 44 px e sem rolagem horizontal em 375 px.

### 4.9 Dados de demonstração

Preparar ao menos uma conta de demonstração com foto, fundo, bio e 3 ou mais avaliações, para o perfil e o arquétipo aparecerem completos no pitch.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Perfil público e histórico
- **Dado** um usuário com avaliações em datas distintas
- **Quando** um visitante abre `/users/{id}`
- **Então** vê nome, data de entrada, bio, imagens e as avaliações da conta, da mais recente para a mais antiga, cada uma com o título do filme e link para a ficha
- **E** um perfil sem avaliações mostra o estado vazio, e um `id` inexistente mostra a página não encontrada.

### Cenário 2: Arquétipo
- **Dado** um usuário com 3 ou mais avaliações vinculadas
- **Quando** qualquer visitante abre o perfil
- **Então** vê o título e a explicação do arquétipo
- **E** um usuário com menos de 3 avaliações mostra "O arquétipo aparece a partir de 3 avaliações."

### Cenário 3: Spoiler oculto no perfil
- **Dado** um histórico com uma avaliação marcada como spoiler (autor ou IA) e outra com 10 ou mais votos de spoiler
- **Quando** um visitante abre o perfil
- **Então** as duas aparecem com o texto oculto e "Mostrar análise", e o título do filme e a nota continuam visíveis.

### Cenário 4: Editar perfil
- **Dado** o titular autenticado em seu próprio perfil
- **Quando** altera bio e avatar e salva
- **Então** o perfil exibe os novos valores
- **E** em outro perfil, ou sem sessão, o botão **Editar perfil** não existe.

### Cenário 5: Foto e fundo
- **Dado** o editor aberto pelo titular
- **Quando** envia uma foto válida e um fundo válido
- **Então** a foto aparece no perfil e o fundo no banner, também para quem visita
- **E** remover a foto volta ao avatar escolhido, remover o fundo volta ao padrão, e um arquivo recusado (tipo errado ou acima de 8 MiB) mantém a imagem anterior com a mensagem de erro visível.

## 6. Definição de Pronto (DoD)

- [ ] `/users/[id]` funcionando contra a API local e contra o deploy de teste (23/11).
- [ ] Arquétipo exibido com e sem dados.
- [ ] Edição, foto e fundo funcionando com a sessão da SPEC-FRONT-07.
- [ ] 8 avatares em SVG com seletor no editor.
- [ ] `apiClient` com corpo binário e `204`, com testes.
- [ ] Conta de demonstração com foto, fundo, bio e 3 ou mais avaliações.
- [ ] Testes ao lado do código e `npm run check` e `npm run build` passando no CI.
- [ ] Documentação criada em `logbuster-web/docs/features/perfil.md`.
- [ ] Status alterado de `Draft` para `Approved` após revisão.
