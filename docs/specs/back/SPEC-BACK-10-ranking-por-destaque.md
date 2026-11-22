# SPEC-BACK-10 — Ranking de Filmes por Destaque da Comunidade

- **Status:** Draft
- **Card(s) Trello:** `[BACK-10]` *(atualizar com o link do card)*
- **Repositório:** `logbuster-api`
- **Responsável:** Marcelo Palmeira (Back A)
- **Dependências:** [`SPEC-BACK-01`](SPEC-BACK-01-post-cadastro-avaliacao.md) (destaques da avaliação), [`SPEC-BACK-04`](SPEC-BACK-04-get-busca-filmes-omdb.md) (formato dos itens de filme), [`SPEC-BACK-05`](SPEC-BACK-05-nota-media-filmes.md) (nota média), [`SPEC-API-00`](../transversais/SPEC-API-00-contrato-http-e-erros.md)
- **Consumido por:** [`SPEC-FRONT-11`](../front/SPEC-FRONT-11-destaques-da-comunidade-home.md) (Marcos)
- **Entrega:** 23/11/2026 (o front depende deste endpoint)

---

## 1. Contexto e Objetivo

Toda avaliação já grava até 2 destaques (Trilha Sonora, Fotografia, Roteiro, Atuação, Direção, Efeitos Especiais), mas esse dado hoje quase não volta para quem lê. Este endpoint devolve os **filmes mais destacados pela comunidade em um aspecto**, para a tela inicial permitir descobrir filmes pela qualidade em que são fortes ("filmes elogiados pela trilha sonora").

Não há tabela nova nem migration: é uma consulta de contagem sobre a coluna `highlights` das avaliações, enriquecida com título e pôster da OMDb.

## 2. Escopo

- **Inclui:** um endpoint público de ranking por destaque, com contagem, nota média, título, ano e pôster de cada filme, e cache de detalhes da OMDb.
- **Não inclui:** ranking ponderado por proporção, combinação de vários destaques, filtro por período, paginação, busca textual combinada e filtro de avaliações dentro de um filme.

## 3. Regras de Negócio

- **RN-01 (Contagem).** Para o destaque pedido, contar, por `movie_id`, quantas avaliações incluem aquele destaque. Todas as avaliações contam, inclusive as marcadas como spoiler e as anônimas antigas, porque o destaque não revela a trama.
- **RN-02 (Ordenação).** Ordenar por quantidade de destaques (decrescente). Desempate: maior nota média do LogBuster, depois `movie_id` em ordem alfabética, para o resultado ser estável.
- **RN-03 (Limite).** Parâmetro `limit` de 1 a 20, padrão 10.
- **RN-04 (Dados do filme).** Cada item traz título, ano e pôster da OMDb, mais a nota média e a contagem de avaliações do LogBuster (mesma fonte da busca, `ratings_map`).
- **RN-05 (Cache e falha da OMDb).** Os detalhes da OMDb ficam em cache no processo (mesmo padrão de `services/movie_titles.py`), com timeout curto (2 s) por filme. Se a OMDb falhar para um filme, o item vem com `title` igual ao próprio `id`, `year` nulo e `poster_url` nulo, sem derrubar a resposta.
- **RN-06 (Vazio).** Sem nenhuma avaliação com aquele destaque, responde `200` com `items: []`.
- **RN-07 (Destaque inválido).** Valor fora do enum `Highlight` responde `422` (`VALIDATION_ERROR`).

## 4. Contrato Técnico

### 4.1 Rota

`GET /api/v1/highlights/{highlight}/movies?limit=10` (público, sem autenticação)

- `highlight`: um de `soundtrack`, `cinematography`, `screenplay`, `acting`, `direction`, `visual_effects`.

> A rota fica sob `/highlights` para não conflitar com `GET /api/v1/movies/{movie_id}`.

### 4.2 Resposta

Os campos de cada item seguem o formato dos itens de `GET /api/v1/movies` (busca), mais `highlight_count`, para o front reaproveitar o mesmo mapper e o mesmo card.

```json
{
  "highlight": "soundtrack",
  "items": [
    {
      "id": "tt0816692",
      "title": "Interstellar",
      "year": "2014",
      "poster_url": "https://…",
      "rating_avg": 4.6,
      "rating_count": 9,
      "highlight_count": 7
    }
  ]
}
```

| Campo | Observação |
| :--- | :--- |
| `highlight_count` | Avaliações que destacaram o aspecto pedido |
| `rating_avg` / `rating_count` | Nota média e total de avaliações do LogBuster |
| `title` / `year` / `poster_url` | Da OMDb. Em falha, `title = id` e os outros nulos (RN-05) |

### 4.3 Implementação sugerida

- **Repositório:** consulta agrupada, por exemplo:
  `SELECT movie_id, count(*) AS highlight_count FROM reviews WHERE :highlight = ANY(highlights) GROUP BY movie_id`, juntando a nota média para o desempate, com `ORDER BY` e `LIMIT` da RN-02 e RN-03.
- **Service:** junta a contagem, a nota (`ratings_map`) e os detalhes da OMDb em cache.
- **Controller:** novo arquivo ou rota em `controllers/`, seguindo o padrão das rotas existentes, com validação do enum e do `limit`.
- Documentar no OpenAPI e no Docsify da API (`docs/endpoints/`).

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Ranking
- **Dado** avaliações em três filmes, com 5, 3 e 1 destaques em "soundtrack"
- **Quando** alguém chama `GET /api/v1/highlights/soundtrack/movies`
- **Então** recebe os três filmes nessa ordem, com `highlight_count`, nota, título, ano e pôster.

### Cenário 2: Desempate
- **Dado** dois filmes com a mesma contagem e notas médias diferentes
- **Quando** o ranking é consultado
- **Então** o de maior nota vem primeiro.

### Cenário 3: Limite
- **Dado** 15 filmes com o destaque
- **Quando** a chamada usa `limit=5`
- **Então** vêm só os 5 primeiros. Sem `limit`, vêm 10. `limit=0` ou `limit=21` responde `422`.

### Cenário 4: Vazio e inválido
- **Dado** nenhum filme com o destaque "visual_effects"
- **Quando** o ranking é consultado
- **Então** responde `200` com `items: []`
- **E** `GET /api/v1/highlights/iluminacao/movies` responde `422`.

### Cenário 5: OMDb indisponível
- **Dado** a OMDb fora do ar
- **Quando** o ranking é consultado
- **Então** responde `200`, e os itens trazem `title` igual ao `id`, com `year` e `poster_url` nulos.

## 6. Definição de Pronto (DoD)

- [ ] Endpoint com testes cobrindo os cenários acima, com a OMDb simulada, e CI verde.
- [ ] OpenAPI e Docsify do `logbuster-api` atualizados.
- [ ] Testado no deploy de teste (23/11).
- [ ] Contrato confirmado com o Marcos (SPEC-FRONT-11).
- [ ] PR revisado e na `main`.
