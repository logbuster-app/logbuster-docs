# LogBuster Docs — Especificações SDD & ADRs

Repositório central de **Specification-Driven Development (SDD)**, contratos entre Front-end e Back-end, Identidade Visual e **Architecture Decision Records (ADRs)** do projeto **LogBuster**.

## Por que um repositório separado (`logbuster-docs`)?

O LogBuster é dividido em três repositórios complementares:

| Repositório | Papel |
| :--- | :--- |
| **`logbuster-docs`** *(este repo)* | Fonte única da verdade transversal: Escopo, Fluxo SDD, Templates, **SPECs** (Front, Back, Contratos HTTP, Identidade Visual) e **ADRs**. |
| **`logbuster-web`** | Aplicação Front-end em Next.js (App Router) + documentação local de código/componentes. |
| **`logbuster-api`** | API Back-end em FastAPI (Python) + documentação local de execução/módulos. |

Manter SPECs transversais e ADRs em `logbuster-docs` evita acoplar decisões do Back-end ou do Produto dentro do repositório do Front-end, permitindo que qualquer card do Trello aponte para um link único e neutro.

## Como visualizar localmente (Docsify)

```bash
npx --yes docsify-cli serve docs -p 3002
```

Acesse `http://localhost:3002` no navegador.
