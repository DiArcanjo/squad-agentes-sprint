# Conexão Roadmap → Paperclip — Especificação (esqueleto)

> **Status: NÃO INICIADA.** Este é um esqueleto de spec. Registra o problema, o objetivo pretendido e as
> questões em aberto conhecidas. Nada aqui é design fechado — decisões serão tomadas quando a feature entrar
> em Specify de fato. Não inventar solução; o que não está definido está marcado como aberto.

## Problem Statement

Hoje a Diana entrega a iniciativa manualmente como task no Paperclip. A intenção é que o **roadmap** (onde a
iniciativa já é priorizada) **empurre a iniciativa para o Paperclip via API**, fechando o ciclo desde a
priorização até o refinamento sem passo manual de entrada. A Diana já criou a feature no roadmap para gerar
uma key. O obstáculo conhecido é de **rede**: o roadmap está hospedado na nuvem (Vercel) e o Paperclip roda em
`localhost`, então não há caminho de rede direto entre os dois.

## Goals (pretendidos, a confirmar)

- [ ] Roadmap empurra uma iniciativa priorizada para o Paperclip via API, sem entrada manual.
- [ ] A iniciativa chega ao Paperclip no formato esperado pelo PO (três blocos: `## Iniciativa`, `## PRD`, `## Documentação técnica`).
- [ ] Resolver a barreira de rede entre roadmap (nuvem/Vercel) e Paperclip (localhost).

## Out of Scope (provisório)

| Item | Motivo |
| ---- | ------ |
| Mudanças no fluxo interno PO → Backlog Manager | Coberto pela feature 001; esta feature só trata da entrada. |

---

## Assumptions & Open Questions

Nenhuma decisão de design foi tomada. Itens abaixo são as pendências conhecidas — todas em aberto.

| Questão | Situação | Confirmado? |
| ------- | -------- | ----------- |
| Direção do fluxo | Pretendida: roadmap **empurra** (push) para o Paperclip via API | n |
| Barreira de rede localhost ↔ Vercel | Precisa de túnel ou exposição pública do Paperclip; abordagem não escolhida | n |
| Formato/contrato do payload roadmap→Paperclip | Não especificado | n |
| Autenticação da API (a key já gerada) | Key gerada no roadmap; uso/escopo não especificado | n |
| Gatilho do push (quando o roadmap dispara) | Não especificado | n |

**Open questions:** todas as acima. Ver também a seção "Questões em aberto" do `.specs/STATE.md`.

---

## User Stories

_A definir quando a feature entrar em Specify. Não preencher por conta própria._

---

## Requirement Traceability

| Requirement ID | Story | Phase | Status |
| -------------- | ----- | ----- | ------ |
| _(a definir)_  | —     | —     | Não iniciada |

---

## Success Criteria

- [ ] _A definir._ Critério mínimo pretendido: uma iniciativa priorizada no roadmap aparece como task no Paperclip sem intervenção manual.
