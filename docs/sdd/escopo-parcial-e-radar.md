# Escopo da Entrega Parcial (10/11/2026) & Radar Pós-Parcial

- **Status:** Approved
- **Card Trello:** [`LB-001`](https://trello.com/c/VHT9kZQV) — *Validar recorte do MVP e entregáveis da disciplina na entrega parcial*
- **Validação de Escopo:** João Raphael (Gestor de Projeto) junto à Equipe Alpha (07/11/2026)

---

## 1. Objetivo da Entrega Parcial (10/11/2026)

Entregar o **Épico de Filmes** funcionando de ponta a ponta como fatia vertical do **LogBuster**, validando a tese central do produto: permitir buscar filmes, consultar os detalhes e a nota média da comunidade, cadastrar avaliações estruturadas (estrelas, destaques e flag provisória de spoiler) e **interagir/ordenar avaliações pela qualidade do argumento (`Bem argumentada`)**.

---

## 2. O que ENTRA na Entrega Parcial (In Scope)

### Front-end (`logbuster-web` — Next.js)
1. **Setup e Identidade Visual (`FRONT-00` / `SPEC-UI-01`):** Arquitetura por features, tokens inspirados na experiência do Letterboxd com marca própria em tema escuro + roxo profundo, CI com cobertura mínima e Docsify.
2. **Busca de Filmes (`FRONT-01` → `FRONT-05`):** Fase 1 consultando a OMDb direto no Front (*concluído*) e Fase 2 migrando o consumo para o endpoint do nosso Back-end (`BACK-04`/`BACK-05`), atualizando a documentação na virada.
3. **Tela de Filme Específico (`FRONT-02` → `FRONT-06`):** Exibição de metadados da obra (Fase 1 via OMDb no Front; Fase 2 via `BACK-07` incluindo nota média calculada pelo Back-end).
4. **Lista, Ordenação e Interação com Avaliações (`FRONT-03`):** Listagem das avaliações do filme vindas do Back-end, seletor de ordenação (com **"Bem argumentada"** como padrão) e botões de interação **`LIKE`**, **`SPOILER`** e **`BEM ARGUMENTADA`**, além de ocultação visual das análises marcadas com spoiler (reveláveis localmente pelo leitor).
5. **Modal de Cadastro de Avaliação (`FRONT-04`):** Estrelas de **`0,5` a `5,0`** (passo `0,5`), até **2 pontos de destaque**, nome/apelido do autor, descrição textual e **checkbox provisório de spoiler**.

### Back-end (`logbuster-api` — FastAPI)
1. **Setup do Repositório (`BACK-00`):** Estrutura modular em camadas, CORS, persistência relacional, tratamento padronizado de erros, CI (`pytest` com cobertura mínima) e Docsify.
2. **Avaliações e Interações (`BACK-01`, `BACK-02`, `BACK-03`, `BACK-06`):** Cadastro (`POST`), listagem ordenada por filme (`GET`), leitura unitária (`GET`) e toggle de interações `like`, `spoiler` e `well_argued` (`POST`).
3. **Catálogo de Filmes via OMDb e Nota Média (`BACK-04`, `BACK-05`, `BACK-07`):** Busca paginada na OMDb enriquecida com nota média do LogBuster, cálculo de média por filme e consulta de detalhes de filme específico na OMDb com nota média agregada.

---

## 3. O que NÃO ENTRA nesta Entrega Parcial (Out of Scope — 10/11)

> **Regra estrita de SDD:** Nenhum item abaixo deve ser incluído nas SPECs ou PRs da entrega parcial de 10/11.

- Detecção/análise automática de spoiler por IA.
- Resumo por IA do consenso das avaliações.
- Tag de spoiler *inline* no meio do texto (estilo Reddit).
- Cadastro de conta, login/autenticação completa, perfil de usuário, listas e amizades.
- Catálogo e acompanhamento de séries/temporadas/episódios.
- Gamificação/troféus e descoberta por percentual de aspectos.
- **Fora do produto inteiro (nem parcial, nem final):** Moderação administrativa e Quiz.

---

## 4. Radar Técnico Pós-Parcial (Backlog Futuro Registrado)

Itens mapeados pela gestão que **ficam fora da parcial de 10/11**, mas já possuem diagnóstico técnico registrado para priorização nas próximas sprints:

### RADAR-01 — Ordenação Geral de Filmes (*Mais novos, Mais populares, Mais bem avaliados*)
- **Desejo de produto:** Permitir ordenar a vitrine/busca geral de filmes por *mais novos*, *mais populares* e *mais bem avaliados*.
- **Restrição técnica mapeada (Alerta OMDb):**
  1. O endpoint gratuito de busca da OMDb (`?s=termo&page=N`) **não suporta parâmetro de ordenação** na origem, retorna apenas lotes de 10 itens e traz somente `Title`, `Year`, `imdbID`, `Type` e `Poster` (sem `imdbVotes` nem `imdbRating`).
  2. A OMDb não possui rota de descoberta/catálogo aberto sem termo de busca (recurso nativo em APIs como a TMDB).
- **Caminhos para quando o item sair do radar:**
  - **Opção A (Com dados da comunidade LogBuster, mantendo OMDb):** Persistir um espelho leve dos filmes avaliados no banco do `logbuster-api` (`imdb_id`, `title`, `year`, `poster_url`) e ordenar tanto a Home quanto páginas buscadas usando métricas próprias do LogBuster (`review_count` para *Mais populares*, `rating_avg` para *Mais bem avaliados* e `year` para *Mais novos*).
  - **Opção B (Migração do adaptador externo para TMDB):** Como `logbuster-web` e `logbuster-api` isolam o formato externo atrás de mappers de domínio, trocar o provider de catálogo no Back-end caso seja necessário catálogo global de populares sem busca textual.

### RADAR-02 — Evolução dos Rótulos Completos de Voto Qualificado
- Expandir na entrega final os rótulos completos da visão macro (*Bem argumentada*, *Concordo*, *Me fez ver de outro jeito*) e regras definitivas de quórum/ocultação coletiva de spoilers quando o módulo de contas/perfis entrar.

---

## 5. Escopo de Back-end da Entrega de 25/11/2026

O planejamento abaixo cobre a Parte 2 do LogBuster no `logbuster-api`.

### [Back A](https://trello.com/c/161rtpgG/81-back-a) — Marcelo Palmeira, com Front 1

[SPEC-BACK-08](../specs/back/SPEC-BACK-08-auth-perfil-publico.md)

- **Cadastro e login (obrigatórios).** Cadastro com nome, e-mail e senha. Login por e-mail e senha com JWT. Novas avaliações são vinculadas à conta quando há token válido. Sem token, a avaliação segue anônima pelo fluxo atual. Avaliações e interações antigas permanecem anônimas, sem migração. Login social, recuperação de senha e confirmação de e-mail ficam fora. Validar cadastro, autenticação e a publicação nos dois fluxos.
- **Perfil público (obrigatório).** Consulta sem login, com nome, data de entrada, bio, avatar, arquétipo quando disponível e histórico paginado das avaliações vinculadas, das mais recentes às mais antigas. Cada avaliação aponta para a ficha do filme. Somente o titular autenticado pode editar bio, avatar, foto e fundo. Seguir pessoas, mensagens e perfil privado ficam fora.
- **Personalização.** A versão mínima de bio e avatar fixo foi ampliada com foto de perfil e fundo enviados pelo titular. A foto tem prioridade sobre o avatar do catálogo enquanto estiver publicada. O fundo usa o padrão do front após a remoção. Temas e links externos ficam fora.
- **Parâmetros implementados.** JWT de uma hora por padrão, senha de 8 a 128 caracteres, nomes de exibição repetíveis, e-mail restrito à sessão, bio de até 160 caracteres e oito IDs de avatar. Formatos, limites e integração das imagens constam na [SPEC-BACK-08](../specs/back/SPEC-BACK-08-auth-perfil-publico.md).

### [Back B](https://trello.com/c/adnU5yAW/82-back-b) — João Victor, com Front 2 e Front 3

[SPEC-BACK-09](../specs/back/SPEC-BACK-09-ia-e-arquetipo.md)

- **Resumo da comunidade com IA (obrigatório, com Front 2).** A partir de 3 avaliações do filme, gerar um parágrafo sob demanda e guardar em cache por filme até o limiar de novas avaliações. Excluir da geração avaliações marcadas como spoiler pelo autor ou ocultas por votos da comunidade. Exibir o texto sem revelar o final, com data e aviso de geração por IA. Abaixo do mínimo, ocultar o bloco. Em caso de falha, demora ou ausência de conteúdo elegível, manter a ficha funcional e sinalizar indisponibilidade. Resumos por pessoa, vários idiomas e perguntas livres ficam fora.
- **Sugestão de spoiler com IA (obrigatória, com Front 2).** Aviso antes de publicar, com opção para o autor marcar ou publicar como está. Falha ou demora permite publicar sem aviso. Mantém-se a ocultação inicial por marcação do autor ou por 3 ou mais votos da comunidade ([regra existente](../specs/transversais/SPEC-API-00-contrato-http-e-erros.md)). A IA não aplica a marcação, não bloqueia a publicação e não reanalisa avaliações antigas.
- **Arquétipo do cinéfilo (opcional, incluído, com Front 3).** Retornar ao perfil um título e uma explicação curta quando houver pelo menos 3 avaliações vinculadas à conta. Abaixo disso, indicar ausência de arquétipo. Recalcular a partir da nota média, destaques mais escolhidos, uso de spoiler e votos recebidos. Teste de personalidade, ranking entre pessoas e escolha manual ficam fora.
- **Máquina do Tempo (condicional, liderada por Front 3 com apoio de Back B).** Fazer o corte em 22/11 conforme o avanço do front. Se aprovada, depende do login e do vínculo usuário–avaliações do Back A. Uma única consulta agrega total de filmes avaliados, nota média, destaque mais escolhido e filme mais bem avaliado. Preparar conta com dados de demonstração. Compartilhar como imagem, comparar com amigos e escolher o período ficam fora.
- **Decisões em aberto.** Quem fornece a chave da IA, modelo, limiar de novas avaliações para regenerar o resumo, momento da análise de spoiler (sugestão de analisar ao tentar publicar), catálogo de arquétipos (sugestão de 5 a 6), regra de cada um e confirmação da pessoa do front responsável pela Máquina do Tempo.

Listas de filmes, recomendações da comunidade, Tinder de filmes, grupos públicos e o filtro de voto por motivo ficam fora deste recorte. A versão mini “mais elogiado por” fica registrada como item separado da ficha do filme.

**Marcos.** 19/11 para início e decisões sobre UI-01 e chave/modelo da IA, 20/11 para desenvolvimento, 22/11 para o corte da Máquina do Tempo, 23/11 para deploy de teste com PostgreSQL e chaves da IA, 24/11 para congelamento de funcionalidades, 25/11 para correções, dados de demonstração e entrega, e 26/11 para pitch e deploy final.
