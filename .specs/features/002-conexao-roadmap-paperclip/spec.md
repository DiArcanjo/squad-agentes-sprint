# Conexão Roadmap → Paperclip — Especificação

> **Natureza deste documento**: especificação de comportamento de agentes (não código-fonte), no mesmo
> padrão adaptado da feature 001. A validação é **funcional e manual** na UI do Paperclip. Esta feature
> está em **Specify** (desenho fechado com a Diana em 2026-07-14); a construção segue em dois **increments**.

## Problem Statement

Hoje a Diana entrega a iniciativa **manualmente** como task no Paperclip (colando os blocos no corpo). A
iniciativa, porém, já nasce priorizada no **Roadmap**. O objetivo é fechar o ciclo priorização → refinamento
**sem passo manual de entrada**, preservando as identidades já validadas (PO decompõe/propõe; Backlog Manager
materializa) e o gate humano.

Duas realidades descobertas ao inspecionar o Roadmap (docs aprovados: `Roadmap/docs/Spec-MCP-Server.md`,
`Roadmap/docs/Integracao-Paperclip.md`) reformataram o desenho:

1. **O Roadmap expõe um servidor MCP, consumido por PULL** — não é o Roadmap que empurra (push). Quem fala MCP
   é o **runtime do agente** (Claude Code) que o Paperclip executa; ele chama a URL pública do Roadmap
   (`…/api/mcp`, Vercel, HTTPS) com uma API key `rmcp_`. Isso **dissolve a barreira de rede** localhost↔Vercel:
   o agente sai para a nuvem, o Roadmap não precisa alcançar o localhost.
2. **Não existe PRD no Roadmap** (ADR-025 do Roadmap eliminou o PRD). Uma iniciativa tem só um campo de
   **Contexto** (`descricao` = o quê/porquê). Como o método de decomposição do PO é ancorado em PRD, é preciso
   um papel que **crie o PRD** a partir do Contexto — e, quando a demanda for técnica, uma **fonte de verdade
   técnica** (código/doc), que no Scrum mora no time de desenvolvimento, não no PO.

Daí o pipeline de **quatro agentes**.

## Goals

- [ ] Um agente **Analista de Requisitos** puxa do Roadmap (MCP) as iniciativas **priorizadas e sem sprint** e cria o **PRD** a partir do Contexto.
- [ ] O **PO** passa a receber o PRD do **Analista** (não mais da Diana manualmente) e decompõe com o método já existente, sem mudança de identidade.
- [ ] **Nenhum agente escreve de volta no Roadmap.** A marcação de sprint é humana; o dedup do Analista vem da própria memória, não de escrita no Roadmap.
- [ ] Barreira de rede resolvida por desenho: o cliente MCP (runtime do agente) chama a Vercel por HTTPS; sem túnel nem exposição do localhost.
- [ ] (Increment 2) Um agente **Tech Lead** fornece, sob demanda, o AS-IS/TO-BE técnico **ancorado** em doc/repo reais; fora do alcance dele, PARA e reporta.

## Pipeline-alvo

```
Roadmap (Contexto, via MCP)
      │  listar_iniciativas {priorizada:true, semSprint:true} → obter_contexto {codigo}
      ▼
Analista de Requisitos  — cria o PRD a partir do Contexto
      │  precisa de contexto técnico?
      │     ├─ não → entrega PRD
      │     └─ sim → consulta Tech Lead (Increment 2) → integra o AS-IS/TO-BE técnico → entrega PRD
      ▼
PO de Refinamento  — decompõe o PRD em Epic→Feature→PBI→Task Dev (skill Decomposição de Backlog)
      │  handoff após aprovação da Diana (gate humano)
      ▼
Backlog Manager  — materializa no Azure Boards
```

**Sequência de construção (AD-012):** Increment 1 = a linha principal (Analista → PO → BM). Increment 2 = o
Tech Lead como consulta técnica sob demanda. Não construir os quatro de uma vez — o wake/handoff do Paperclip
já é frágil (questão em aberto herdada da 001) e cada agente novo multiplica os pontos de travamento.

## Out of Scope

| Item | Motivo |
| ---- | ------ |
| Fluxo interno PO → Backlog Manager | Coberto pela feature 001; aqui só muda o **produtor** da entrada do PO. |
| Escrita de volta no Roadmap (marcar sprint, `atribuir_sprint`) | Decisão: marcação de sprint é **humana** na UI do Roadmap (AD-016). Nenhum agente do Paperclip escreve no Roadmap. |
| Mudanças de código no Roadmap | O servidor MCP já existe e está aprovado; a integração é **config-only** (key + `.mcp.json`). |
| Modelagem de sprint como entidade | `sprint` é texto livre na iniciativa (ADR-031 do Roadmap), fora do nosso escopo. |
| Método de decomposição do PO | Inalterado (skill Decomposição de Backlog). Só a **fonte** do PRD muda. |
| Wake pós-aprovação 100% confiável | Questão em aberto herdada da 001; não resolvida aqui. |

---

## Assumptions & Open Questions

| Assunção / decisão | Situação | Racional | Confirmado? |
| ------------------ | -------- | -------- | ----------- |
| Direção do fluxo | **PULL via MCP** (o runtime do agente é o cliente; chama `…/api/mcp`) — não push | `Roadmap/docs/Integracao-Paperclip.md` | **y** |
| Barreira de rede localhost↔Vercel | **Dissolvida por desenho**: o agente chama a Vercel por HTTPS; o Roadmap não alcança o localhost | idem | **y** |
| Contrato do payload | As 8 tools do MCP do Roadmap; a fila = `listar_iniciativas {priorizada:true, semSprint:true}`, conteúdo via `obter_contexto` | `Roadmap/docs/Spec-MCP-Server.md` | **y** |
| Autenticação | Key `rmcp_` como Secret no Paperclip; `.mcp.json` (escopo projeto) com `Authorization: Bearer` | `Integracao-Paperclip.md` §2–3 | **y** |
| Gatilho | Ao ser **ativado** (wake), o Analista checa a fila `priorizada + semSprint`. Sem intervalo (mantém AD-009) | decisão Diana 14/07 | **y** |
| Fonte do PRD | Não existe PRD no Roadmap; o **Analista** cria o PRD a partir do Contexto | ADR-025 do Roadmap + decisão Diana | **y** |
| Dono do AS-IS/TO-BE | Migra do PO para o **Analista** (negócio sempre; técnico via Tech Lead). PO só **consome** | aderência ao Scrum (PO = o quê/porquê; técnico = time) — AD-015 | **y** |
| Dedup do Analista | Auto-dedup por **memória própria** (histórico no Paperplip): não recria PRD para um código já tratado. Sem escrita no Roadmap | decisão Diana; respeita "ninguém escreve de volta" — AD-016 | **y** |
| Papel do Analista no Roadmap | **Read-only** (papel `gerente_projeto` do Roadmap, que é só-leitura por `exigirEscrita`/ADR-018) | contém privilégio; coerente com "não escreve" | n (validar na criação da key) |
| Fonte técnica do Tech Lead | Recebe um doc **ou** acessa um repo para validar; **fora do alcance → PARA e reporta** | decisão Diana 14/07 — AD-014 | **y** |
| Reference name do campo do PRD entre Analista→PO | Formato do handoff Analista→PO a definir na construção (Increment 1) | — | n |
| Wake pós-aprovação/ativação confiável | Não confiável hoje; *Run Heartbeat* manual força (herdado da 001) | STATE.md | n |

**Open questions:** ver também a seção "Questões em aberto" do `.specs/STATE.md`.

---

## User Stories

### P1: Analista puxa a fila e cria o PRD ⭐ MVP (Increment 1)

**User Story**: Como **Analista de Requisitos** (agente), quero, ao ser ativado, puxar do Roadmap as iniciativas priorizadas e sem sprint e criar um PRD a partir do Contexto de cada uma, para que o PO tenha um PRD para decompor sem entrada manual da Diana.

**Why P1**: É a nova entrada do sistema. Sem ela, o ciclo roadmap→refinamento não fecha.

**Acceptance Criteria**:

1. WHEN o Analista é ativado (wake) THEN SHALL chamar `listar_iniciativas { priorizada: true, semSprint: true }` no MCP do Roadmap e tratar o resultado como a **fila** a refinar.
2. WHEN há uma iniciativa na fila THEN o Analista SHALL obter o conteúdo dela via `obter_contexto { codigo }` (identificação por **código**, ex.: `FIN-06`).
3. WHEN o Analista tem o Contexto THEN SHALL produzir um **PRD** (requisitos + critérios de aceite de negócio; AS-IS/TO-BE de negócio) e entregá-lo ao PO.
4. WHEN o Contexto está vazio, insuficiente ou ambíguo THEN o Analista SHALL parar e reportar à Diana — sem inventar requisito.
5. WHEN o Analista acessa o Roadmap THEN SHALL fazê-lo **somente em leitura** (key `rmcp_` de papel read-only) — nunca escrever (não chama `atribuir_sprint`, `atualizar_status`, `mover_periodo` nem `gravar_esforco_agente`).

**Independent Test** (funcional, manual): com uma iniciativa priorizada e sem sprint no Roadmap, ativar o Analista e verificar que ele lê o Contexto, produz um PRD e o entrega ao PO, sem escrever nada no Roadmap.

---

### P1: Analista evita reprocessar (dedup por memória) ⭐ MVP (Increment 1)

**User Story**: Como Analista, quero não recriar o PRD de uma iniciativa que já tratei, para que a fila `semSprint` (que só é limpa quando o humano marca a sprint, tarde) não me faça reprocessar a mesma iniciativa a cada ativação.

**Why P1**: Sem dedup, toda ativação no intervalo entre decomposição e marcação humana de sprint gera trabalho duplicado.

**Acceptance Criteria**:

1. WHEN o Analista pega um `codigo` da fila THEN SHALL verificar no próprio histórico (Paperclip) se já produziu PRD para esse `codigo`.
2. WHEN já existe PRD para o `codigo` THEN o Analista SHALL pular a iniciativa e NÃO recriar o PRD.
3. WHEN o dedup é feito THEN SHALL ser por memória própria — o Analista NÃO escreve no Roadmap para sinalizar "já tratei".

**Independent Test** (funcional, manual): ativar o Analista duas vezes com a mesma iniciativa ainda sem sprint e confirmar que o PRD é criado só na primeira.

---

### P1: PO recebe o PRD do Analista ⭐ MVP (Increment 1)

**User Story**: Como PO de Refinamento, quero receber o PRD do Analista (em vez de a Diana colar a task), para decompor com o método atual sem mudar minha identidade.

**Why P1**: É o encaixe do PO no novo pipeline. A mudança é só de **produtor** da entrada.

**Acceptance Criteria**:

1. WHEN o Analista entrega um PRD THEN o PO SHALL tratá-lo como sua entrada e decompor em Epic→Feature→PBI→Task Dev (skill Decomposição de Backlog), como na feature 001.
2. WHEN a entrada do PO é descrita THEN SHALL refletir que o **produtor é o Analista**, não a Diana — a Entrada em `po/AGENTS.md` e `po/HEARTBEAT.md` deixa de exigir "task com três blocos colada pela Diana".
3. WHEN o PO recebe o PRD THEN o AS-IS/TO-BE (inclusive técnico, quando houver) já vem **no PRD** — o PO **consome**, não produz o AS-IS/TO-BE técnico (AD-015).
4. WHEN falta um elemento essencial do PRD THEN o PO SHALL parar e reportar (regra herdada da 001), agora sobre o PRD do Analista.

**Independent Test** (funcional, manual): entregar um PRD do Analista ao PO e verificar que ele decompõe igual à 001, sem produzir AS-IS/TO-BE técnico por conta própria.

---

### P2: Tech Lead fornece AS-IS/TO-BE técnico ancorado (Increment 2)

**User Story**: Como Tech Lead (agente), quero fornecer, sob demanda, o estado técnico atual e o alvo (AS-IS/TO-BE técnico) ancorado em doc ou repositório reais, para que o PRD do Analista tenha base técnica confiável quando a demanda for técnica.

**Why P2**: Fecha o furo do "de onde vem o AS-IS técnico" sem sobrecarregar o Analista nem o PO. Não é mainline: só demandas técnicas.

**Acceptance Criteria**:

1. WHEN o Analista roteia uma consulta técnica ao Tech Lead THEN o Tech Lead SHALL responder ancorado em fonte real — um doc fornecido e/ou um repositório que ele acessa para validar.
2. WHEN o sistema em questão está **fora do alcance** do Tech Lead (sem doc e sem repo acessível) THEN SHALL **parar e reportar** ("não tenho o contexto técnico deste sistema") — nunca inventar AS-IS.
3. WHEN o Tech Lead responde THEN o **Analista** SHALL integrar o AS-IS/TO-BE técnico no PRD (dono único do PRD); o Tech Lead não entrega direto ao PO.

**Independent Test** (funcional, manual): com uma demanda técnica e um repo acessível, verificar que o Tech Lead produz um AS-IS ancorado no código; com um sistema fora do alcance, verificar que ele para e reporta.

---

### P2: Analista roteia para o Tech Lead quando precisa (Increment 2)

**User Story**: Como Analista, quero rotear ao Tech Lead apenas quando a demanda é técnica e eu não tenho o contexto, para que a consulta técnica seja sob demanda e não um hop obrigatório.

**Acceptance Criteria**:

1. WHEN a demanda é de negócio/processo OU o Analista já tem o contexto técnico THEN SHALL seguir direto para o PO, sem acionar o Tech Lead.
2. WHEN a demanda é técnica E o Analista não tem o contexto THEN SHALL criar uma consulta na fila do Tech Lead e aguardar a resposta antes de fechar o PRD.

**Independent Test** (funcional, manual): uma demanda de negócio não aciona o Tech Lead; uma técnica sem contexto aciona.

---

### P2: Contenção e credencial do Analista

**User Story**: Como responsável pela segurança, quero que o Analista carregue só o privilégio do seu papel (leitura do Roadmap), para conter o estrago mesmo com skip permissions.

**Acceptance Criteria**:

1. WHEN o Analista é configurado THEN a key do Roadmap SHALL ser **read-only** (papel `gerente_projeto`) e injetada como **Secret** do Paperclip, referenciada no `.mcp.json` (nunca em texto plano).
2. WHEN as cercas de contenção são aplicadas THEN o Analista SHALL seguir as mesmas da 001 (heartbeat sem intervalo, create-agents OFF, import-skills OFF, max concurrent 1 — AD-009).

**Independent Test** (funcional, manual): auditar a config do Analista — key read-only, Secret referenciado, cercas ativas — e complementar com teste funcional (o vetor real é `Bash`/MCP, não só a auditoria — aprendizado da 001).

---

## Edge Cases

- WHEN a fila `priorizada + semSprint` vem vazia THEN o Analista encerra limpo, sem trabalho (não inventa iniciativa).
- WHEN o mesmo `codigo` reaparece na fila (ainda sem sprint) após já ter PRD THEN o Analista pula (dedup, CRP-04..06).
- WHEN a auth do MCP falha (key inválida/revogada → 401) THEN o Analista PARA e reporta — não busca outra credencial.
- WHEN o Roadmap está inacessível (rede/Vercel fora) THEN o Analista PARA e reporta.
- WHEN o Contexto é insuficiente para um PRD THEN o Analista PARA e reporta (não inventa requisito).
- WHEN o Tech Lead não alcança o sistema THEN PARA e reporta (não inventa AS-IS).
- WHEN o Paperclip não reacorda o Analista/PO de forma confiável THEN *Run Heartbeat* manual força (herdado da 001).

---

## Requirement Traceability

| Requirement ID | Story | Increment | Phase | Status |
| -------------- | ----- | --------- | ----- | ------ |
| CRP-01 | P1: Analista puxa a fila (listar_iniciativas priorizada+semSprint) | 1 | Execute | Repo ✓ / Paperclip pendente |
| CRP-02 | P1: Analista obtém conteúdo (obter_contexto por código) | 1 | Execute | Repo ✓ / Paperclip pendente |
| CRP-03 | P1: Analista cria o PRD (negócio) e entrega ao PO | 1 | Execute | Repo ✓ / Paperclip pendente |
| CRP-04 | P1: Analista para/reporta com Contexto insuficiente | 1 | Execute | Repo ✓ / Paperclip pendente |
| CRP-05 | P1: Analista read-only no Roadmap (não escreve) | 1 | Execute | Repo ✓ / Paperclip pendente |
| CRP-06 | P1: Analista dedup por memória (não recria PRD) | 1 | Execute | Repo ✓ / Paperclip pendente |
| CRP-07 | P1: PO recebe o PRD do Analista (Entrada: produtor = Analista) | 1 | Execute | Repo ✓ / Paperclip pendente |
| CRP-08 | P1: PO consome AS-IS/TO-BE do PRD, não produz o técnico | 1 | Execute | Repo ✓ / Paperclip pendente |
| CRP-09 | P2: Tech Lead fornece AS-IS/TO-BE técnico ancorado | 2 | Specify | Pendente |
| CRP-10 | P2: Tech Lead para/reporta fora do alcance | 2 | Specify | Pendente |
| CRP-11 | P2: Analista integra a resposta do Tech Lead no PRD | 2 | Specify | Pendente |
| CRP-12 | P2: Analista roteia ao Tech Lead só quando técnico e sem contexto | 2 | Specify | Pendente |
| CRP-13 | P2: Contenção/credencial do Analista (read-only, Secret, cercas) | 1 | Execute | Repo ✓ / Paperclip pendente |

**ID format:** `CRP-[NUMBER]` (Conexão Roadmap-Paperclip).

**Status values:** Pendente → Em Design → Repo ✓ / Paperclip pendente → Validado.

**Coverage:** 13 requisitos. **Increment 1 (mainline) — CRP-01..08, CRP-13 — escrito no repo** (arquivos do Analista em `agentes/analista-requisitos/`; Entrada do PO e DoR da skill ajustadas). Falta: sincronizar no Paperclip (criar o agente Analista, colar os 4 arquivos, gerar a key `rmcp_` read-only, configurar `.mcp.json`, re-importar a skill por *install-update*) e o **teste funcional manual**. **Increment 2 (Tech Lead): CRP-09..12** — ainda não construído.

---

## Success Criteria

- [ ] Uma iniciativa priorizada e sem sprint no Roadmap vira um PRD (Analista) → decomposição (PO) → materialização (BM), sem entrada manual da Diana.
- [ ] Nenhum agente do Paperclip escreve no Roadmap; a marcação de sprint permanece humana.
- [ ] O Analista não reprocessa uma iniciativa já tratada.
- [ ] (Increment 2) Demandas técnicas recebem AS-IS/TO-BE ancorado do Tech Lead; sistemas fora do alcance param e reportam.
- [ ] Barreira de rede não bloqueia (o agente chama a Vercel por HTTPS).
