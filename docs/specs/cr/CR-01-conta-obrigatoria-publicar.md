# CR-01 — Conta obrigatória para publicar avaliação

- **Status:** Approved
- **Card Trello:** `[CR-01]` *(atualizar com o link do card)*
- **Repositório:** `logbuster-api` (back) e `logbuster-web` (front, no card FRONT-07)
- **Responsável:** Marcelo Palmeira (Back A), com Thyago Viana no front
- **Aprovação:** João Raphael (Gestor de Projeto), 22/11/2026
- **Prazo:** 23/11/2026 (entra no deploy de teste)
- **Altera:** [`SPEC-BACK-01`](../back/SPEC-BACK-01-post-cadastro-avaliacao.md) (cadastro de avaliação), [`SPEC-BACK-08`](../back/SPEC-BACK-08-auth-perfil-publico.md) (RN-02 e cenário 2) e [`SPEC-FRONT-07`](../front/SPEC-FRONT-07-contas-sessao-perfil.md) (RN-01, RN-07 e cenários 4 e 5)

---

## 1. Motivo

Com a publicação anônima, qualquer pessoa pode publicar quantas avaliações quiser com nomes diferentes (*review bombing*), sem nenhum vínculo de identidade. Além disso, tudo o que a Parte 2 entrega (perfil, histórico, arquétipo e Máquina do Tempo) depende de a avaliação estar ligada a uma conta.

> Exigir conta aumenta a barreira, mas não elimina contas múltiplas, porque não há confirmação de e-mail. Confirmação de e-mail e limite de requisições ficam registrados como próximos passos.

## 2. O que muda (delta)

| Antes (SPEC original) | Depois (este CR) |
| :--- | :--- |
| `POST /api/v1/reviews` aceita publicar sem token, com `author_name` informado | Sem token válido, responde `401 UNAUTHORIZED` e nada é salvo |
| O nome do autor vem de `author_name` (anônimo) ou da conta (logado) | O nome do autor vem **sempre** da conta |
| O modal de avaliação tem modo anônimo, com campo de nome | O modal só abre com sessão. Sem sessão, "Avaliar filme" leva ao login |
| Sessão expirada no modal → passa ao modo anônimo | Sessão expirada no modal → pede login de novo, preservando o texto |

## 3. Regras de Negócio

### Back (`logbuster-api`)

- **RN-01 (Token obrigatório).** `POST /api/v1/reviews` usa a dependência `RequiredUserId` em vez de `OptionalUserId`. Sem token, ou com token inválido ou vencido, responde `401` com `error.code = UNAUTHORIZED` e não persiste nada.
- **RN-02 (Autor vem da conta).** A avaliação é sempre criada pelo fluxo do usuário autenticado, com vínculo à conta e nome de exibição da conta. O ramo anônimo do controller é removido.
- **RN-03 (`author_name` ignorado).** O campo continua aceito no payload, para não quebrar clientes que ainda o enviem, mas é **ignorado**. O schema usa `extra="forbid"`, então remover o campo faria esses clientes receberem `422`.
- **RN-04 (Avaliações antigas preservadas).** As avaliações anônimas já existentes continuam no banco e aparecem normalmente nas listagens, médias e resumo, sem atribuição retroativa.
- **RN-05 (Votos não mudam aqui).** As interações (curtir, spoiler e bem argumentada) continuam por `X-Visitor-Id`. A mudança delas é do CR-03.

### Front (`logbuster-web`, card FRONT-07)

- **RN-06 (Avaliar exige sessão).** Sem sessão, o botão **"Avaliar filme"** continua visível e leva para `/login?next=/movies/{id}`. Depois do login, a pessoa volta para o filme.
- **RN-07 (Modal sem nome).** O modal não tem mais o campo de nome. Mostra "Publicando como *nome*" e não envia `author_name`. O nome salvo em `localStorage` (`logbuster:review_author_name`) deixa de ser usado.
- **RN-08 (Sessão expirada no modal).** Se a publicação retornar `401`, o modal **preserva** nota, destaques, texto e marcação de spoiler e mostra um aviso com o botão "Entrar de novo". Depois do login, a pessoa volta ao filme. Guardar o rascunho para reabri-lo é desejável, mas não obrigatório.

## 4. Contrato Técnico

| Situação | Resposta |
| :--- | :--- |
| `POST /api/v1/reviews` sem `Authorization` | `401` `UNAUTHORIZED` |
| Com token inválido ou vencido | `401` `UNAUTHORIZED` |
| Com token válido e payload válido | `201`, avaliação vinculada à conta, `author_name` da resposta igual ao nome de exibição da conta |
| Com token válido e `author_name` no payload | `201`, e o valor enviado é ignorado |

Atualizar a descrição da rota no OpenAPI: "Requer token Bearer. O nome do autor vem da conta."

## 5. Fora do Escopo

Confirmação de e-mail, limite de requisições, migração das avaliações anônimas para contas, votos com conta (CR-03).

## 6. Critérios de Aceite (Gherkin)

### Cenário 1: Publicar sem conta
- **Dado** um visitante sem token
- **Quando** chama `POST /api/v1/reviews` com um payload válido
- **Então** recebe `401` e nenhuma avaliação é salva.

### Cenário 2: Publicar com conta
- **Dado** uma conta com token válido
- **Quando** publica uma avaliação, com ou sem `author_name` no payload
- **Então** recebe `201`, a avaliação fica vinculada à conta e exibe o nome da conta.

### Cenário 3: Avaliações antigas
- **Dado** avaliações anônimas criadas antes deste CR
- **Quando** a lista do filme, a média e o resumo são consultados
- **Então** elas continuam aparecendo e sendo contadas.

### Cenário 4: Front sem sessão
- **Dado** um visitante sem sessão na ficha de um filme
- **Quando** clica em "Avaliar filme"
- **Então** vai para o login e, depois de entrar, volta para o mesmo filme.

### Cenário 5: Sessão expira durante o preenchimento
- **Dado** o modal aberto com texto digitado
- **Quando** a publicação retorna `401`
- **Então** o texto e as seleções permanecem e aparece o aviso para entrar de novo.

## 7. Definição de Pronto (DoD)

- [ ] Back: rota exigindo token, ramo anônimo removido, testes atualizados (os testes que publicam sem login passam a usar token) e CI verde.
- [ ] Back: OpenAPI e Docsify do `logbuster-api` atualizados.
- [ ] Front: comportamento das RN-06 a RN-08 entregue no card FRONT-07.
- [ ] Testado no deploy de teste (23/11).
- [ ] Status deste CR alterado para `Implemented` no [registro](README.md).
- [ ] PR revisado e na `main`.
