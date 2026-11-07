# Fluxo SDD, Revisão, ADRs e Definição de Pronto (DoD)

> **Rastreabilidade Trello:** Atende aos cards [`LB-005`](https://trello.com/c/Stmyha6J) (*Adotar template de SPEC, ADR e fluxo de revisão*) e [`LB-007`](https://trello.com/c/IbH6y1hv) (*Definir responsáveis e execução do teste manual de cada épico*).

---

## 1. Como funciona o ciclo SDD no LogBuster

Todo desenvolvimento segue a sequência **SPEC → Implementação + Testes → PR + Revisão → ADR (pós-PR, quando aplicável)**:

1. **Antes de codar (Definition of Ready — DoR):**
   - O card no Trello possui link preenchido no campo **Spec de referência documentada: Sim** apontando para `logbuster-docs/docs/specs/...`.
   - Os contratos HTTP (rotas, payloads e erros) e cenários Gherkin estão lidos pelo responsável.
2. **Durante a implementação:**
   - Se o backend ainda não estiver disponível para uma task de front, o dev implementa primeiro com mock fiel ao contrato da SPEC (PR-1) e conecta ao endpoint real em seguida (PR-2), ou entrega integrado quando o endpoint já estiver na `main`.
   - Qualquer mudança de contrato descoberta na prática **exige ajuste na SPEC** para manter Front e Back sincronizados.
3. **No Pull Request (Definition of Done — DoD de todo card):**
   - [ ] **Código** cobrindo 100% dos critérios de aceite da SPEC.
   - [ ] **Testes automatizados** passando no CI respeitando a cobertura mínima configurada.
   - [ ] **Docsify local do repositório** (`logbuster-web/docs` ou `logbuster-api/docs`) atualizado com a feature/endpoint entregue (especialmente quando uma integração provisória mudar de Front para Back).
   - [ ] **Revisão cruzada** aprovada por pelo menos 1 integrante diferente do autor.
4. **Após o Merge dos PRs — Registro de ADRs (`logbuster-docs/docs/adr/`):**
   - Para **não travar a velocidade dos PRs** em janelas curtas de entrega, as **ADRs são documentadas após o merge dos PRs** que consolidaram decisões arquiteturais ou escolhas técnicas relevantes (ex.: banco/ORM adotado no `BACK-00`, detalhes da migração OMDb Front → Back, estratégia de identificação sem login).
   - Use sempre o [Template de ADR](ADR-TEMPLATE.md).

---

## 2. Governança de Aprovação e Teste Manual (`LB-005` e `LB-007`)

| Atividade | Responsável | Regra |
| :--- | :--- | :--- |
| **Aprovação de Escopo e SPECs** | João Raphael (Gestor) | Valida alinhamento entre Front, Back e escopo da entrega parcial. |
| **Revisão de PRs — Front (`logbuster-web`)** | Revisão cruzada entre Marcos, Elton e Thyago | `FRONT-01` revisado por Elton e Thyago; demais cards revisados por no mínimo 1 par do Front. |
| **Revisão de PRs — Back (`logbuster-api`)** | Revisão cruzada entre Marcelo e João Victor | Marcelo revisa PRs de `reviews`; João Victor revisa PRs de `movies`/setup. |
| **Teste Manual Integrado do Épico de Filmes (`LB-007`)** | João Raphael + 1 dev Front + 1 dev Back (em **09/11** / **10/11**) | Executa o roteiro ponta a ponta abaixo antes da demo parcial. |

### Roteiro de Smoke Test Manual Integrado (Entrega Parcial — 10/11)

1. Abrir a Home (`/`), buscar um filme real (ex.: *"Matrix"*), validar loading e grid de resultados.
2. Testar busca sem resultados (ex.: *"xyzfilmeinesistente999"*) e validar estado vazio amigável.
3. Clicar em um filme da grade para abrir `/movies/[id]` e validar metadados e nota média inicial.
4. Abrir o modal de nova avaliação, tentar enviar inválido (sem estrelas ou com 3 destaques) e validar bloqueio visual.
5. Cadastrar avaliação válida com estrelas (`0,5` a `5,0` em passos de `0,5`), até 2 destaques, descrição e checkbox de spoiler marcado.
6. Confirmar atualização da lista de avaliações na tela do filme, ocultação inicial do texto marcado como spoiler (com botão para revelar localmente) e recálculo da nota média do filme.
7. Interagir com avaliações usando **`LIKE`**, **`SPOILER`** e **`BEM ARGUMENTADA`**, recarregar a página e validar persistência dos contadores e a **ordenação por "Bem argumentada"** (e demais filtros de ordenação).
