# Change Requests (CR) — Registro de Mudanças

Registro das mudanças aprovadas **depois** que uma SPEC foi aprovada ou implementada.

## Regras

1. **A SPEC original não é editada.** Ela continua como foi aprovada, para manter o histórico do que foi decidido e entregue.
2. **Cada CR descreve só o delta:** o que muda, em qual SPEC e em qual seção, e o motivo.
3. **Comportamento vigente = SPEC original + CRs com status `Implemented`, aplicados em ordem numérica.** Em conflito, vale o CR mais recente.
4. Cada CR tem um card no Trello com o mesmo código (`[CR-NN]`) e segue o mesmo fluxo de revisão e DoD das SPECs ([Fluxo SDD](../../sdd/fluxo-sdd-e-dod.md)).
5. Status possíveis: `Proposed` → `Approved` → `Implemented` (ou `Rejected`).

## Registro

| CR | Título | SPECs afetadas | Responsável | Status |
| :--- | :--- | :--- | :--- | :--- |
| CR-01 | Conta obrigatória para publicar avaliação | SPEC-BACK-01, SPEC-BACK-08, SPEC-FRONT-07 | Marcelo | Approved *(arquivo a escrever)* |
| [CR-02](CR-02-spoiler-ia-pos-publicacao.md) | Spoiler com IA processado depois de publicar | SPEC-BACK-09, SPEC-API-00, SPEC-BACK-01, SPEC-BACK-02, SPEC-BACK-03, SPEC-BACK-08 | João Victor | Approved |
| CR-03 | Conta obrigatória para votar e reagir | SPEC-BACK-06, SPEC-FRONT-03 | Marcelo | Approved *(arquivo a escrever)* |
