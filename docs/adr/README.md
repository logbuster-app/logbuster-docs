# Architecture Decision Records (ADRs)

Este diretório armazena as **ADRs** transversais do projeto **LogBuster** em um repositório neutro (`logbuster-docs`), separado do código específico do Front (`logbuster-web`) e do Back (`logbuster-api`).

## Política: Documentar ADRs após os PRs

Para manter agilidade na entrega parcial de **10/11/2026** sem perder governança técnica:

1. **SPECs vêm ANTES do código** (`docs/specs/`) para alinhar contrato entre Front e Back.
2. **ADRs vêm APÓS o merge dos PRs** que consolidam decisões de arquitetura, infraestrutura ou trade-offs relevantes descobertos na implementação — usando o [ADR-TEMPLATE](../sdd/ADR-TEMPLATE.md).
3. Não exigimos ADR para PRs puramente funcionais que apenas seguem uma SPEC existente sem introduzir nova decisão estrutural.

---

## Índice de ADRs

### Consolidadas
- **[ADR-001 — Stack Tecnológica e Divisão em Três Repositórios](ADR-001-stack-e-tres-repositorios.md)** *(Consolidada após `LB-003` e PR #1 do `logbuster-web`)*

### Fila prevista para registro pós-PRs (Entrega Parcial)
- **`ADR-002` — Arquitetura Interna, ORM e Banco do `logbuster-api`** *(Registrar após merge do PR de `BACK-00` pelo Marcelo)*
- **`ADR-003` — Transição da Integração OMDb do Front-end para o Back-end** *(Registrar após merge dos PRs de `BACK-04`/`BACK-07` e `FRONT-05`/`FRONT-06`)*
- **`ADR-004` — Identificação de Visitante sem Login e Regras Provisórias de Voto/Spoiler no MVP Parcial** *(Registrar após merge dos PRs de `BACK-01`/`BACK-06` e `FRONT-03`/`FRONT-04`)*
