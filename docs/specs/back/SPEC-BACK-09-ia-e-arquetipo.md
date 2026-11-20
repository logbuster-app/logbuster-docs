# SPEC-BACK-09 — Resumo da comunidade, sugestão de spoiler e arquétipo

- **Status:** Draft
- **Card(s) Trello:** [Back B](https://trello.com/c/adnU5yAW/82-back-b)
- **Repositório:** `logbuster-api`
- **Responsável:** João Victor, com Front 2 (IA) e Front 3 (arquétipo e Máquina do Tempo)
- **Dependências:** [SPEC-API-00](../transversais/SPEC-API-00-contrato-http-e-erros.md), [SPEC-BACK-01](SPEC-BACK-01-post-cadastro-avaliacao.md), [SPEC-BACK-06](SPEC-BACK-06-post-interacao-avaliacao.md), [SPEC-BACK-08](SPEC-BACK-08-auth-perfil-publico.md) para recursos da conta
- **Entrega:** 25/11/2026. Corte da Máquina do Tempo em 22/11/2026.

## 1. Contexto e Objetivo

Resumir a opinião da comunidade na ficha do filme e ajudar o autor a identificar possíveis spoilers antes de publicar. Usar o histórico da conta para derivar o arquétipo no perfil. Se confirmada no corte, apoiar a retrospectiva do Front 3.

## 2. Escopo

- **Obrigatório:** Resumo da comunidade gerado sob demanda e em cache, e sugestão de spoiler antes de publicar.
- **Opcional incluído:** Arquétipo calculado a partir de avaliações. **Condicional:** Máquina do Tempo, se Front 3 estiver adiantado em 22/11.
- **Fora do escopo:** Resumos personalizados, vários idiomas e perguntas livres. Bloqueio, marcação automática pela IA e reanálise de avaliações antigas. Teste de personalidade, ranking e escolha manual de arquétipo. Escolha de período, comparação entre amigos e compartilhamento como imagem da Máquina do Tempo.

## 3. Regras de Negócio

- **RN-01.** O resumo só aparece a partir de 3 avaliações do filme. O limiar inicial conta avaliações do filme. O conteúdo usa somente avaliações com `is_spoiler == false` e `counts.spoiler < 3`. Entregar um parágrafo sobre a opinião da comunidade, sem revelar o final ou outros spoilers, com data para a identificação “Gerado por IA”.
- **RN-02.** Gerar sob demanda e guardar por filme. Reutilizar a geração válida até o limiar acordado de novas avaliações, com uma geração em andamento por filme. Um resumo baseado em avaliação que passa a ser ocultada por spoiler fica indisponível até uma próxima geração elegível. Quando não há texto elegível, ou em caso de timeout, falha ou resposta imprópria da IA, sinalizar indisponibilidade para o bloco e preservar a ficha.
- **RN-03.** A análise de spoiler recebe somente o rascunho e o ID do filme. Havendo suspeita, o front avisa e o autor escolhe marcar ou publicar como está. Em falha ou demora da IA, o fluxo publica normalmente, sem aviso. A análise funciona também para autores anônimos e não cria, altera nem oculta avaliações. A regra de ocultação inicial permanece por marcação do autor ou por 3 ou mais votos de spoiler da comunidade.
- **RN-04.** A partir de 3 avaliações vinculadas à conta, recalcular um título e uma explicação curta do arquétipo com base na nota média, destaques mais escolhidos, uso de spoiler e votos recebidos. Abaixo do mínimo, o perfil indica “ainda sem arquétipo”. O resultado deriva dos dados, sem cadastro manual.
- **RN-05.** A Máquina do Tempo depende da decisão de 22/11, do login e do vínculo da avaliação com a conta. Se aprovada, o Front 3 lidera e o Back B oferece uma única consulta de agregação com total de filmes avaliados, média, destaque mais escolhido e filme mais bem avaliado. Preparar conta com dados de demonstração.
- **Decisões em aberto:** Quem fornece a chave de IA, modelo, quantidade de novas avaliações que regenera o resumo e momento da análise no modal (proposta de analisar ao tentar publicar). A equipe define o catálogo de arquétipos (sugestão de 5 a 6), regras e precedência. No corte de 22/11, confirmar avanço e responsável do front. Se entrar, fechar a janela fixa da retrospectiva e os critérios de agregação e desempate com Front 3.

## 4. Contrato Técnico

Proposta para revisão com Front 2 e Front 3. Rotas com JSON em `snake_case`, IDs de filme IMDb e envelope de erros da SPEC-API-00.

| Método e rota | Entrada e autenticação | Resposta |
| :--- | :--- | :--- |
| `GET /api/v1/movies/{movie_id}/review-summary` | Público. `movie_id` no padrão IMDb | `200` com `{status, text, generated_at}`. A consulta usa as avaliações locais e funciona independentemente da chamada da ficha |
| `POST /api/v1/reviews/spoiler-check` | Público. `{movie_id, content}`. O texto usa a mesma validação da SPEC-BACK-01 | `200` com `{status}`. O front decide se mostra o aviso. O `POST /api/v1/reviews` recebe a escolha em `is_spoiler` |
| `GET /api/v1/users/{user_id}` | Público, rota definida na SPEC-BACK-08 | Campo `archetype` nulo antes de 3 avaliações ou objeto `{title, description}` calculado com os dados da conta |
| `GET /api/v1/users/me/time-machine` | Somente se aprovada. `Authorization: Bearer <jwt>`. Período fixo, sem parâmetro de seleção | `200` com `{rated_movies_count, rating_avg, top_highlight, top_movie_id}`. Sem avaliações no período, total `0` e demais campos `null` |

O resumo usa `status: "ready"` quando `text` e `generated_at` estão disponíveis, `"insufficient_reviews"` abaixo do limiar inicial e `"unavailable"` diante de falha ou ausência de conteúdo elegível. Nos dois últimos casos, `text` e `generated_at` são nulos. O front oculta o bloco apenas em `insufficient_reviews` e mostra mensagem simples em `unavailable`.

```json
{
  "status": "ready",
  "text": "A comunidade destaca a direção e a trilha sonora do filme.",
  "generated_at": "2026-11-23T15:00:00Z"
}
```

Na sugestão de spoiler, `status` é `"suspected"`, `"clear"` ou `"unavailable"`. Somente `"suspected"` aciona o aviso. A rota de publicação não depende de uma chamada anterior à IA.

```json
{
  "movie_id": "tt0133093",
  "content": "O texto digitado pelo autor no modal de avaliação."
}
```

```json
{
  "status": "suspected"
}
```

Exemplo de arquétipo no perfil, com título ilustrativo:

```json
{
  "archetype": {
    "title": "Crítico exigente",
    "description": "Costuma reservar as notas mais altas para poucos filmes."
  }
}
```

Se aprovada, a retrospectiva conta filmes distintos, calcula a média das avaliações no período, usa o enum de `highlights` e identifica o filme de maior nota com `movie_id`:

```json
{
  "rated_movies_count": 7,
  "rating_avg": 4.2,
  "top_highlight": "direction",
  "top_movie_id": "tt0133093"
}
```

| HTTP | `error.code` | Aplicação |
| :--- | :--- | :--- |
| `401` | `UNAUTHORIZED` | JWT ausente ou inválido na retrospectiva |
| `404` | `NOT_FOUND` | Usuário inexistente ao consultar perfil (SPEC-BACK-08) |
| `422` | `VALIDATION_ERROR` | ID malformado ou texto inválido |
| `500` | `INTERNAL_ERROR` | Erro interno inesperado da API. Falhas do provedor de IA usam o estado `unavailable` |

## 5. Critérios de Aceite (Gherkin)

### Cenário 1 — Filme abaixo do mínimo
- **Dado** um filme com duas avaliações
- **Quando** o resumo é solicitado
- **Então** retorna `insufficient_reviews` sem consultar a IA.

### Cenário 2 — Filtro de spoilers e cache
- **Dado** um filme com três ou mais avaliações, incluindo uma marcada pelo autor e outra com três votos de spoiler
- **Quando** há texto elegível e o resumo é solicitado
- **Então** a IA recebe só os textos elegíveis. A API retorna texto sem spoilers e data, reutiliza o cache enquanto válido e regenera sob demanda ao atingir o limiar de novas avaliações.

### Cenário 3 — Falha ou perda de elegibilidade do resumo
- **Dado** um filme acima do mínimo com erro ou timeout na IA, sem nenhum texto elegível, ou com resumo em cache que usou uma avaliação recém-ocultada por spoiler e ainda não tem nova geração elegível
- **Quando** o front solicita o bloco
- **Então** o resumo indisponível retorna `unavailable`, sem revelar texto antigo ou inelegível. A consulta de detalhes do filme continua funcional.

### Cenário 4 — Sugestão, escolha e quórum da comunidade
- **Dado** um rascunho que a IA classifica como possível spoiler
- **Quando** a análise é solicitada e o autor decide publicar
- **Então** a análise retorna `suspected` e a publicação persiste a escolha do autor em `is_spoiler`.
- **E** a avaliação fica inicialmente oculta ao atingir três votos de spoiler, mesmo se o autor escolheu publicar sem marcar.

### Cenário 5 — Falha na sugestão
- **Dado** um rascunho válido e a IA indisponível ou demorando
- **Quando** o autor tenta publicar
- **Então** a análise sinaliza `unavailable` e a avaliação pode ser criada normalmente, sem aviso de spoiler.

### Cenário 6 — Cálculo de arquétipo
- **Dado** um perfil com duas avaliações vinculadas
- **Quando** o perfil é consultado, recebe uma terceira avaliação e é consultado novamente
- **Então** o arquétipo passa de `null` para um título e uma explicação de acordo com as regras aprovadas.
- **E** alterações nos dados usados nas regras aparecem no próximo cálculo.

### Cenário 7 — Retrospectiva condicional
- **Dado** que a equipe aprovou o recurso no corte, com uma conta autenticada e avaliações no período de demonstração
- **Quando** essa conta solicita a retrospectiva
- **Então** recebe os quatro indicadores calculados em uma agregação com os dados da própria conta.
- **E** requisição sem JWT recebe `401`. Sem aprovação no corte, o cenário fica fora da entrega.

## 6. Definição de Pronto (DoD)

- [ ] Validar contrato de IA com Front 2 e contrato de arquétipo com Back A e Front 3. Registrar o corte da retrospectiva.
- [ ] Implementar cache, filtro de spoilers, sugestão com publicação independente e cálculo de arquétipo. Entregar a agregação apenas se aprovada no corte.
- [ ] Cobrir limiares, cache, ocultação, falhas de IA e agregações com testes automatizados e cobertura mínima no CI.
- [ ] Validar a configuração no deploy de teste de 23/11. Atualizar OpenAPI e o Docsify em `logbuster-api/docs/` e obter revisão cruzada no PR.
- [ ] Vincular esta SPEC ao card Back B.
