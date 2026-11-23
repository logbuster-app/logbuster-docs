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
| [CR-01](CR-01-conta-obrigatoria-publicar.md) | Conta obrigatória para publicar avaliação | SPEC-BACK-01, SPEC-BACK-08 *(front incorporado direto na SPEC-FRONT-07)* | Marcelo | Implemented na `main` (`045f8cf`, 22/11), falta validar no deploy de teste |
| [CR-02](CR-02-spoiler-ia-pos-publicacao.md) | Antispoiler: IA após publicar e comunidade com 10 votos | SPEC-BACK-09, SPEC-BACK-01, SPEC-API-00, SPEC-FRONT-03 | João Victor, com Marcos | Implemented na `main` (`d5344b5`, 23/11) e validado com IA real (10/10), falta validar no deploy de teste e o front usar 10 votos |
| CR-03 | Conta obrigatória para votar e reagir | SPEC-BACK-06, SPEC-FRONT-03 | Marcelo | Approved *(arquivo a escrever)* |
| CR-04 | Antispoiler inteligente: reavaliação escalada da IA por votos, comunidade decide quando a IA não tem contexto (sem sinopse ou filme recente), peso de voto por reputação | SPEC-BACK-09, SPEC-BACK-06 | — | Proposed *(próximo passo, fora da entrega de 25/11)* |
