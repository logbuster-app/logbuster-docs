# SPEC-BACK-08 — Autenticação e perfil público

- **Status:** Implemented (As-Built no back-end)
- **Card(s) Trello:** [Back A](https://trello.com/c/161rtpgG/81-back-a)
- **Repositório:** `logbuster-api`
- **Responsável:** Marcelo Palmeira, com Front 1
- **Dependências:** [SPEC-API-00](../transversais/SPEC-API-00-contrato-http-e-erros.md), [SPEC-BACK-01](SPEC-BACK-01-post-cadastro-avaliacao.md), [SPEC-BACK-02](SPEC-BACK-02-get-lista-avaliacoes.md)
- **Entrega:** 25/11/2026

## 1. Contexto e Objetivo

Permitir que novas avaliações sejam associadas a uma conta e apareçam no perfil público, preservando o fluxo de publicação anônima. O Front 1 integra cadastro, login e perfil. O Front 1 e o Back B consomem o vínculo entre usuário e avaliação.

## 2. Escopo

- **Inclui:** Cadastro com nome de exibição, e-mail e senha, login por JWT e consulta da sessão. Perfil público, histórico paginado e edição de bio, avatar de catálogo, foto de perfil e fundo pelo titular. O arquétipo é calculado na [SPEC-BACK-09](SPEC-BACK-09-ia-e-arquetipo.md) e aparece no perfil quando disponível.
- **Fora do escopo:** Login social, recuperação de senha, confirmação de e-mail, seguir pessoas, mensagens, perfil privado, temas e links externos.

## 3. Regras de Negócio

- **RN-01.** Usar o e-mail como identificador de login, com unicidade na conta. Guardar somente o hash da senha. A API emite JWT assinado com o ID da conta no assunto e expiração ao autenticar credenciais válidas.
- **RN-02.** Com JWT válido, vincular a avaliação nova ao usuário identificado no token e usar seu nome de exibição. Sem cabeçalho de autorização, manter o fluxo com `author_name` informado pelo visitante. Um token inválido ou vencido resulta em `401`. O vínculo é opcional no banco e avaliações e interações antigas continuam anônimas, sem atribuição retroativa. As interações preservam o contrato atual de `X-Visitor-Id`.
- **RN-03.** Qualquer visitante consegue consultar o perfil, com nome, data de entrada, bio, avatar, fundo e arquétipo quando disponível. A lista contém somente avaliações vinculadas à conta, das mais recentes às mais antigas, com referência à ficha de cada filme pelo `movie_id`.
- **RN-04.** Apenas o titular autenticado edita a bio, o avatar fixo e as imagens do perfil. A bio aceita até 160 caracteres. O catálogo aceita os IDs de `avatar-01` a `avatar-08`. O Front 1 associa os IDs às ilustrações.
- **RN-05.** O titular pode enviar, substituir e remover a foto e o fundo independentemente. Quando há foto personalizada, ela tem prioridade sobre o avatar fixo. Ao removê-la, o avatar fixo escolhido volta a ser usado. O fundo assume o padrão do front quando não há imagem personalizada. As imagens são públicas junto com o perfil.
- **RN-06.** Aceitar JPEG, PNG e WebP estáticos de até 8 MiB e até 4096 pixels em cada dimensão. Corrigir a orientação, preservar a proporção e gerar WebP sem metadados, com limite de saída de 512 × 512 para a foto e 1920 × 1080 para o fundo. Um envio rejeitado preserva a imagem anterior.
- **Parâmetros adotados na API.** JWT de 3600 segundos por padrão, configurável via `JWT_EXPIRES_IN`. Senha de 8 a 128 caracteres com hash Argon2id. Nomes de exibição podem se repetir. O e-mail é exibido somente na sessão do titular.

## 4. Contrato Técnico

Contrato para integração com Front 1. As rotas com JSON usam `snake_case` e o envelope de erros da SPEC-API-00. IDs de usuário, avaliação e versão da imagem são UUIDs. Datas são ISO-8601 em UTC. CORS aceita `Authorization` e os métodos `PATCH`, `PUT` e `DELETE` nas origens configuradas.

| Método e rota | Entrada e autenticação | Resposta |
| :--- | :--- | :--- |
| `POST /api/v1/auth/register` | `{display_name, email, password}` | `201` com `{user_id, display_name, created_at}` |
| `POST /api/v1/auth/login` | `{email, password}` | `200` com `{access_token, token_type: "bearer", expires_in}`. Validade em segundos |
| `GET /api/v1/auth/me` | `Authorization: Bearer <jwt>` | `200` com `{user_id, display_name, email, created_at, bio, avatar_id, avatar_image_url, background_image_url}` do titular |
| `GET /api/v1/users/{user_id}` | Público | `200` com o perfil abaixo |
| `GET /api/v1/users/{user_id}/reviews` | Público. `page` a partir de 1 (padrão 1), `page_size` de 1 a 50 (padrão 10) | `200` com `{items, total, page, page_size}`. Itens no formato de SPEC-BACK-02, ordenados por `created_at DESC, id DESC`. Página vazia retorna `items: []` |
| `GET /api/v1/avatars` | Público | `200` com `{items: [{id}]}`. O front associa cada ID à ilustração acordada |
| `PUT /api/v1/users/me/images/{kind}` | Bearer do titular. `kind` é `avatar` ou `background`. Corpo binário com o `Content-Type` da imagem | `200` com `{url}` relativa à origem da API |
| `DELETE /api/v1/users/me/images/{kind}` | Bearer do titular. Remoção idempotente da imagem desse tipo | `204` sem corpo |
| `GET /api/v1/users/{user_id}/images/{kind}/{version}` | Público. A URL é fornecida no perfil ou no PUT | `200` com bytes `image/webp` ou `404` se a versão não estiver disponível |
| `PATCH /api/v1/users/me` | Bearer do titular. `{bio?, avatar_id?}`. Campo omitido preserva o valor. `null` limpa o campo | `200` com o perfil atualizado. O usuário alvo vem do token |
| `POST /api/v1/reviews` | Payload da SPEC-BACK-01. Bearer opcional. `author_name` é obrigatório no fluxo anônimo e pode ser omitido no autenticado, que usa o nome da conta | `201` com o formato atual. Persiste o vínculo apenas quando há JWT válido |

O cadastro aceita, por exemplo:

```json
{
  "display_name": "Pessoa cinéfila",
  "email": "cinefila@example.com",
  "password": "senha-de-exemplo"
}
```

O login usa os campos `email` e `password` do cadastro. A resposta de autenticação identifica o tipo e a validade do token:

```json
{
  "access_token": "<jwt>",
  "token_type": "bearer",
  "expires_in": 3600
}
```

O valor de `expires_in` no exemplo é o padrão da API.

Exemplo de perfil público com foto personalizada e fundo padrão:

```json
{
  "user_id": "c4f7d9d2-9d4e-4b3a-8f55-1f2e3d4c5b6a",
  "display_name": "Pessoa cinéfila",
  "created_at": "2026-11-20T14:00:00Z",
  "bio": "Prefiro ficção científica.",
  "avatar_id": "avatar-01",
  "avatar_image_url": "/api/v1/users/c4f7d9d2-9d4e-4b3a-8f55-1f2e3d4c5b6a/images/avatar/d66cd35e-8a14-4499-9da1-b89936f7e88f",
  "background_image_url": null,
  "archetype": null
}
```

`bio`, `avatar_id`, `avatar_image_url`, `background_image_url` e `archetype` aceitam `null`. O objeto de arquétipo, quando calculado, segue a SPEC-BACK-09. Itens do histórico mantêm `is_spoiler` e `counts.spoiler` para a ocultação inicial na interface.

| HTTP | `error.code` | Aplicação |
| :--- | :--- | :--- |
| `401` | `UNAUTHORIZED` | Credenciais inválidas, JWT inválido ou expirado, ou ausência de autenticação nas rotas privadas |
| `404` | `NOT_FOUND` | Usuário inexistente no perfil ou histórico, ou imagem/versão inexistente |
| `409` | `CONFLICT` | E-mail já cadastrado |
| `413` | `PAYLOAD_TOO_LARGE` | Arquivo acima de 8 MiB ou dimensão superior a 4096 pixels |
| `415` | `UNSUPPORTED_MEDIA_TYPE` | `Content-Type` ausente, não aceito ou diferente do formato detectado |
| `422` | `VALIDATION_ERROR` | Campos inválidos, bio longa, avatar fora do catálogo, paginação inválida ou imagem inválida ou animada |

### Integração das imagens no Front 1

1. Enviar o próprio `File` no corpo do `PUT` com Bearer e o tipo da imagem em `Content-Type`. O envio usa bytes diretos, sem `FormData` nem Base64.
2. Converter a URL relativa em absoluta com `new URL(url, apiOrigin)`. Exibir a URL da foto antes do avatar de catálogo e usar o fundo padrão se o campo de fundo for nulo. O GET da imagem não exige cabeçalho de autorização.
3. Ao trocar uma imagem, usar o `url` devolvido pelo PUT ou recarregar o perfil. Ao remover, chamar o DELETE e recarregar o perfil. Versões removidas ou substituídas retornam `404`. A resposta binária usa `Cache-Control: no-store`.
4. Em erro de envio, manter a imagem atual e apresentar a mensagem do envelope de erro. A alteração de imagem é independente do PATCH de bio e avatar fixo.

```ts
const response = await fetch(new URL(`/api/v1/users/me/images/${kind}`, apiOrigin), {
  method: "PUT",
  headers: { Authorization: `Bearer ${token}`, "Content-Type": file.type },
  body: file,
})
// Em 200, o JSON { url } contém o caminho da versão atual.
```

### Persistência e migração

A migração Alembic `0003` adiciona uma tabela de imagens em PostgreSQL com uma linha por conta e tipo. O modelo guarda somente o WebP normalizado e seu ID de versão. A atualização da mesma imagem do perfil é atômica. Ativar com `uv run --frozen alembic upgrade head`. O perfil retorna URLs nulas para contas que ainda não enviaram imagens.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1 — Cadastro e sessão
- **Dado** um cadastro válido com e-mail novo
- **Quando** a pessoa se cadastra, autentica com suas credenciais e consulta a sessão com o JWT
- **Então** recebe `201` no cadastro e `200` nas consultas autenticadas, com nome e ID da conta. Senha e hash ficam fora das respostas
- **E** tentativa de login com senha incorreta recebe `401`.

### Cenário 2 — Publicação autenticada e anônima
- **Dado** um visitante e uma conta com JWT válido
- **Quando** ambos publicam avaliações válidas e outra chamada tenta publicar com JWT expirado
- **Então** a publicação autenticada fica vinculada à conta, a publicação sem token é aceita como anônima e a chamada com JWT expirado recebe `401` sem persistir a avaliação
- **E** avaliações e interações anteriores ao lançamento permanecem anônimas.

### Cenário 3 — Perfil e histórico públicos
- **Dado** um usuário com avaliações em datas distintas, incluindo um filme que também recebeu avaliações anônimas
- **Quando** um visitante abre o perfil e navega no histórico
- **Então** recebe os dados públicos e somente as avaliações da conta, paginadas da mais recente para a mais antiga, com `movie_id`.
- **E** um perfil sem avaliações retorna histórico vazio e continua acessível.

### Cenário 4 — Edição de bio e avatar
- **Dado** um titular autenticado, outro usuário e um catálogo fixo
- **Quando** o titular envia uma bio dentro do limite e um avatar válido
- **Então** o perfil exibe ambos os valores. Uma sessão de outra conta só pode editar o próprio perfil
- **E** atualização sem sessão recebe `401`. Bio longa demais ou avatar desconhecido recebe `422`, sem alterar o perfil.

### Cenário 5 — Foto e fundo personalizados
- **Dado** que o titular escolheu um avatar fixo e tem uma sessão válida
- **Quando** envia uma foto e um fundo aceitos
- **Então** o perfil público e `/auth/me` expõem as duas URLs. A foto é exibida no lugar do avatar fixo.
- **E** trocar ou remover uma imagem não altera a outra. A URL antiga deixa de funcionar e remover a foto recupera o avatar fixo.

### Cenário 6 — Envio rejeitado mantém o perfil
- **Dado** que uma foto já está publicada
- **Quando** outra pessoa tenta enviar sem autenticação, ou o titular envia um arquivo incompatível, animado ou acima do limite
- **Então** a API retorna o erro correspondente e a URL e a imagem anteriores permanecem disponíveis.

## 6. Definição de Pronto (DoD)

- [ ] Validar a integração das imagens com Front 1 e do campo de arquétipo com Back B.
- [x] Implementar fluxo autenticado e anônimo, perfil, histórico e edição, com vínculo opcional e armazenamento separado de foto e fundo.
- [ ] Cobrir sucesso, validação, autorização, paginação, compatibilidade anônima e ciclo de vida das imagens com testes automatizados e cobertura mínima no CI.
- [x] Atualizar OpenAPI e o Docsify em `logbuster-api/docs/`.
- [ ] Obter revisão cruzada no PR.
- [ ] Vincular esta SPEC ao card Back A.
