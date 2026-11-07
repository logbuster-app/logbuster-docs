# Template de Especificação SDD (`SPEC-TEMPLATE`)

> Copie este modelo ao criar uma nova SPEC em `docs/specs/`.

```markdown
# SPEC-XXX — Título da Funcionalidade ou Endpoint

- **Status:** Draft | Approved | Implemented (As-Built)
- **Card(s) Trello:** [Código do Card](https://trello.com/c/...)
- **Repositório:** `logbuster-web` | `logbuster-api` | Transversal
- **Responsável:** Nome
- **Dependências:** Outras SPECs ou endpoints necessários

---

## 1. Contexto e Objetivo
Breve descrição do problema resolvido por este card dentro do produto.

## 2. Escopo
- **Inclui:** O que será entregue neste card.
- **Não inclui (Fora de escopo):** O que explicitamente NÃO deve ser implementado aqui.

## 3. Regras de Negócio
- **RN-01:** ...
- **RN-02:** ...

## 4. Contrato Técnico
- **Se Back-end:** Método, rota, headers, query/path params, JSON de Request/Response, status HTTP e erros.
- **Se Front-end:** Rotas, componentes/hooks em `src/features/<feature>`, tipos de domínio, estados (`idle`, `loading`, `empty`, `error`, `success`) e tokens visuais.

## 5. Critérios de Aceite (Gherkin)

### Cenário 1: Caminho feliz
- **Dado** ...
- **Quando** ...
- **Então** ...

### Cenário 2: Validação / Erro
- **Dado** ...
- **Quando** ...
- **Então** ...

## 6. Definição de Pronto (DoD)
- [ ] Código implementado seguindo a arquitetura do repositório.
- [ ] Testes automatizados cobrindo sucesso e erro passando no CI (cobertura mínima).
- [ ] Documentação local (Docsify do repositório) atualizada.
- [ ] Card no Trello atualizado com link para esta SPEC.
```
