# STATE

> Memória do projeto **squad-agentes-sprint**: sistema multiagente de refinamento de sprint
> construído e em operação dentro do Paperclip (orquestrador em `localhost:3100`). Este repositório
> é a fonte de verdade da configuração dos agentes, trazida da UI do Paperclip para versionamento.
>
> Adaptação da skill `tlc-spec-driven`: este NÃO é um projeto de código. Não há build, testes
> automatizados nem código-fonte. Usamos as fases **Specify** e o **STATE.md** de decisões; NÃO
> executamos a fase Execute com testes nem o Verifier de mutação. "Implementar" aqui é registrar a
> configuração dos agentes; "validar" é descrever o teste funcional feito manualmente na UI do Paperclip.

## Decisions

### AD-001
- **Decision**: O agente escritor no Azure Boards se chama **Backlog Manager**.
- **Reason**: Soa natural no vocabulário ágil e é estreito o suficiente para conter o escopo (só materializar backlog).
- **Trade-off**: Descartados "Escritor"/"Publicador" (soavam informais) e "Scrum Master" (papel largo demais, carregaria autoridade de processo indesejada justo no agente que detém a credencial de escrita).
- **Scope**: Feature 001 — nomenclatura e identidade do agente escritor.
- **Date**: 2026-07-13
- **Status**: active

### AD-002
- **Decision**: Separação de poderes por agente — o PO só lê (credencial `pmo-read`), o Backlog Manager só escreve (credencial `board-write`).
- **Reason**: Isola privilégios; permite revogar a escrita sem derrubar a leitura, pois as credenciais ficam em secrets separados.
- **Trade-off**: Mais credenciais para gerenciar; handoff explícito entre os dois agentes em vez de um agente único onipotente.
- **Scope**: Feature 001 — modelo de permissões dos dois agentes.
- **Date**: 2026-07-13
- **Status**: active

### AD-003
- **Decision**: A escrita acontece **apenas na sandbox**. `board-write` é escopada a *Work Items Read & Write* na organização `dianasandbox` apenas, com expiração curta.
- **Reason**: Contém o estrago mesmo com skip permissions ligado; nunca toca produção (`technecloud`).
- **Trade-off**: Necessário renovar a credencial com frequência (expiração curta); nenhum caminho automático para produção.
- **Scope**: Feature 001 — escopo da credencial `board-write`.
- **Date**: 2026-07-13
- **Status**: active

### AD-004
- **Decision**: **Gate humano obrigatório**. Nada vai para o Azure sem aprovação da Diana (confirmation `ACCEPTED`). O aceite da decomposição é a autorização de escrita.
- **Reason**: Um humano revisa a árvore Epic→Feature→PBI→Task antes de qualquer materialização; o aceite funciona como assinatura de autorização.
- **Trade-off**: O fluxo não é 100% autônomo — depende de uma ação humana no meio.
- **Scope**: Feature 001 — ponto de controle entre decomposição (PO) e materialização (Backlog Manager).
- **Date**: 2026-07-13
- **Status**: active

### AD-005
- **Decision**: **Assigned To sempre vazio**. O Backlog Manager cria todos os itens no Azure sem atribuir ninguém.
- **Reason**: Atribuição é decisão humana, feita manualmente após o poker de estimativa.
- **Trade-off**: Passo manual adicional depois da materialização; o agente não fecha o ciclo de alocação.
- **Scope**: Feature 001 — comportamento de escrita do Backlog Manager e contrato `decomposicao_json`.
- **Date**: 2026-07-13
- **Status**: active

### AD-006
- **Decision**: **Handoff automático** via instrução no `HEARTBEAT.md` do PO. Ao detectar a aprovação (confirmation `ACCEPTED`), o PO cria automaticamente a task para o Backlog Manager com a árvore aprovada (`decomposicao_json`) copiada inteira no corpo da task (formato do carregamento detalhado em AD-010).
- **Reason**: Evita um passo manual de repasse entre aprovação e materialização; mantém o fluxo fluindo depois do gate humano.
- **Trade-off**: Depende do Paperclip reacordar o PO de forma confiável após a aprovação — o que hoje não é 100% garantido (ver Aprendizados e Questões em aberto).
- **Scope**: Feature 001 — transição PO → Backlog Manager.
- **Date**: 2026-07-13
- **Status**: active

### AD-007
- **Decision**: A credencial é injetada via **secret do Paperclip como env var** (`AZURE_DEVOPS_EXT_PAT`), não via `az login`.
- **Reason**: Descoberta empírica — o `az login` feito no terminal NÃO alcança o runtime do adapter; só a env var injetada funciona.
- **Trade-off**: A credencial vive como variável de ambiente no runtime do adapter (superfície a conter); depende do mecanismo de secrets do Paperclip.
- **Scope**: Feature 001 — mecanismo de injeção de credencial para ambos os agentes.
- **Date**: 2026-07-13
- **Status**: active

### AD-008
- **Decision**: O método de decomposição vive como **arquivo chapado** (`DECOMPOSICAO.md`), não como skill do Paperclip.
- **Reason**: O Paperclip não permite criar pasta na aba de arquivos, e uma skill exige pasta.
- **Trade-off**: Não aproveita o mecanismo de skills do Paperclip (descoberta/versionamento nativo); é apenas um arquivo de instrução lido pelo PO.
- **Scope**: Feature 001 — forma de empacotar o método de decomposição do PO.
- **Date**: 2026-07-13
- **Status**: superseded by AD-011 (2026-07-14)
- **Correção (2026-07-14)**: a premissa do Reason está factualmente errada diante da doc do Paperclip (`/reference/skills/`). Uma skill não precisa ser criada na aba de arquivos — pode ser **importada** (`sourceKind` `github`/`url`/`local_path`/scan de projeto). Como este repo é versionado em git e já contém uma skill em pasta (`tlc-spec-driven`), o método pode virar `SKILL.md` importado do git. Ver AD-011.

### AD-009
- **Decision**: **Cercas de contenção** dos agentes: heartbeat sem intervalo (só *wake on demand*), *Continue after max-turn* OFF, *create-agents* OFF, *create/import-skills* OFF, *max concurrent* = 1.
- **Reason**: A contenção vem de permissão e run policy combinadas, não de um único toggle. Impede que os agentes se auto-expandam ou rodem em loop.
- **Trade-off**: Menos autonomia operacional; wake precisa ser disparado (por evento ou manualmente via Run Heartbeat).
- **Scope**: Feature 001 — run policy e permissões de ambos os agentes no Paperclip.
- **Date**: 2026-07-13
- **Status**: active

### AD-010
- **Decision**: No handoff PO → Backlog Manager, a árvore aprovada viaja **EMBUTIDA** (Forma B): o PO copia o `decomposicao_json` inteiro no corpo da task de handoff, e a **existência dessa task é a autorização** do BM — o BM lê o JSON do próprio corpo, não navega para outra task nem verifica confirmation `ACCEPTED`.
- **Reason**: Torna o BM autossuficiente na task em que acorda. Concentra a confiança da autorização em um único fato verificável (o PO só cria a task de handoff após a Diana aprovar), eliminando a navegação entre tasks e a checagem de aprovação remota — que eram fonte de dessincronização e faziam o BM parar por "sem aprovação".
- **Trade-off**: O `decomposicao_json` fica duplicado (na task de decomposição original e no corpo do handoff); se a Diana editar a decomposição após o handoff, a cópia embutida não reflete a mudança. Descartada a **Forma A** (BM segue ponteiro/ID para a task de decomposição original), que evita a duplicação mas reintroduz a navegação entre tasks e a leitura da aprovação em outro lugar.
- **Scope**: Feature 001 — mecanismo de transporte e autorização do handoff. Refina AD-006.
- **Date**: 2026-07-13
- **Status**: active

### AD-011
- **Decision**: O método de decomposição passa a ser empacotado como **skill do Paperclip no padrão `SKILL.md`** (pasta `skills/decomposicao-backlog/` com frontmatter + `references/`), versionada no git como fonte de verdade, para ser **importada no Paperclip via `sourceKind` git** e atualizada por *install-update* (1 clique). Supera o AD-008. O arquivo chapado `agentes/po/DECOMPOSICAO.md` foi **colapsado num tombstone** (breadcrumb, não vai para as Instructions) após a skill ser importada, anexada ao PO e validada; as citações a ele em `po/SOUL.md`, `po/HEARTBEAT.md` e `po/AGENTS.md` foram repontadas para a skill.
- **Reason**: (1) A premissa do AD-008 (skill exige criação na aba de arquivos) é falsa — skills podem ser importadas do git. (2) Git como fonte única de verdade elimina o copy-paste manual do método para o Paperclip. (3) **Reuso**: o Refinador planejado (validador da DoR) precisa do mesmo método (INVEST/DoR/SPIDR) — uma skill company-level anexável aos dois agentes via `desiredSkills` é o lar natural do "um produz, o outro valida".
- **Trade-off**: (1) O update do git **não é automático** — é *update-status* → *install-update* (1 clique, sem polling); melhor que colar, mas não zero-touch. (2) Só o **método** ganha esse fluxo; os arquivos de definição de agente (`SOUL`/`AGENTS`/`TOOLS`/`HEARTBEAT`) **não** têm git-import na doc do Paperclip e continuam sincronizados **à mão** — decisão consciente da Diana (aceitável, pois esses arquivos são a maior parte do churn mas o ganho de sync não os cobre). (3) O schema `decomposicao_json` fica em mais um lugar (`skills/decomposicao-backlog/references/`) até a duplicação com `contratos/` ser consolidada.
- **Scope**: Feature 001 — forma de empacotar o método de decomposição; substitui AD-008.
- **Date**: 2026-07-14
- **Status**: active
- **Resolvido (2026-07-14)**: a semântica da cerca `create/import-skills OFF` (AD-009) foi confirmada na UI do Paperclip — o **import humano no nível da empresa e o attach ao agente via `desiredSkills` funcionam** com a cerca ligada (a cerca contém o **agente** de auto-importar em runtime, não a Diana). A skill "Decomposição de Backlog" entrou como `sourceKind: github` trackando `main` (SHA `4e54566`, "Up to date") e foi anexada ao PO (Installed skill ativa). O fluxo git → Paperclip por *install-update* (1 clique) está operante para o método.

### AD-012
- **Decision**: A conexão Roadmap→Paperclip (feature 002) é um **pipeline de 4 agentes** — **Analista de Requisitos → (Tech Lead, sob demanda) → PO → Backlog Manager** — construído em **dois increments**: Increment 1 = a linha principal (Analista → PO → BM); Increment 2 = o Tech Lead como consulta técnica sob demanda.
- **Reason**: O Roadmap só expõe **Contexto** (sem PRD — ADR-025 do Roadmap), então é preciso um papel que **crie o PRD** (Analista) e, para demandas técnicas, uma **fonte de verdade técnica** (Tech Lead). Sequenciar em increments contém o risco: o wake/handoff do Paperclip já é frágil e cada agente novo multiplica os pontos de travamento — valida-se a mainline antes do roteamento condicional.
- **Trade-off**: Mais agentes e handoffs (superfície de orquestração maior) sobre um mecanismo de wake ainda não confiável. Mitigado pela sequência (mainline primeiro) e por manter o Tech Lead como **consulta**, não etapa fixa.
- **Scope**: Feature 002 — arquitetura e sequência de construção.
- **Date**: 2026-07-14
- **Status**: active

### AD-013
- **Decision**: O fluxo Roadmap→Paperclip é **PULL via MCP**, não push. Quem fala MCP é o **runtime do agente** (Claude Code) que o Paperclip executa; ele chama a URL pública do Roadmap (`…/api/mcp`, Vercel, HTTPS) com a API key `rmcp_` (Secret no Paperclip, referenciada no `.mcp.json` de escopo projeto). A **barreira de rede** localhost↔Vercel fica **dissolvida por desenho** (o agente sai para a nuvem; o Roadmap não precisa alcançar o localhost).
- **Reason**: Descoberto ao inspecionar `Roadmap/docs/Integracao-Paperclip.md` e `Spec-MCP-Server.md` (docs aprovados): o Paperclip não fala MCP direto; o cliente MCP é o runtime do agente. Isso reverte a premissa "push" do esqueleto antigo da spec 002 e elimina a necessidade de túnel/exposição do localhost.
- **Trade-off**: A key vive como Secret no runtime do adapter (superfície a conter); depende da URL pública do Roadmap estar no ar.
- **Scope**: Feature 002 — direção do fluxo e resolução da barreira de rede. Supera a premissa "push" do esqueleto anterior.
- **Date**: 2026-07-14
- **Status**: active

### AD-014
- **Decision**: Novo agente **Analista de Requisitos** — dono do MCP do Roadmap (**read-only**, papel `gerente_projeto`), cria o **PRD** a partir do Contexto e o entrega ao PO. É quem **puxa** a fila (`listar_iniciativas {priorizada:true, semSprint:true}`) e lê o conteúdo (`obter_contexto {codigo}`). Papel estreito: cria PRD, não decompõe (PO), não prioriza nem marca sprint (humano), não escreve no Azure (BM).
- **Reason**: Preenche a lacuna do PRD sem sobrecarregar o PO nem quebrar sua identidade. Concentra a conexão com o Roadmap num só agente read-only.
- **Trade-off**: Um agente a mais no pipeline; handoff Analista→PO a projetar (Increment 1).
- **Scope**: Feature 002 — Increment 1.
- **Date**: 2026-07-14
- **Status**: active

### AD-015
- **Decision**: O **AS-IS/TO-BE** (inclusive o técnico) migra do PO para o **Analista de Requisitos**, entregue **dentro do PRD**. O PO passa a **consumir** o AS-IS/TO-BE, não produzir o técnico.
- **Reason**: Aderência ao Scrum — o PO é papel de valor/negócio (o quê/porquê); o "como"/estado técnico atual é do time de desenvolvimento, não do PO. A DoR anterior punha "AS-IS/TO-BE técnico" no colo do PO, o que excedia o papel. Com o Analista (papel de análise de requisitos) existindo, o AS-IS/TO-BE é dele.
- **Trade-off**: A skill Decomposição de Backlog (DoR) precisa de ajuste para refletir que o PO **consome** o AS-IS/TO-BE em vez de produzir o técnico (a fazer na construção do Increment 1).
- **Scope**: Feature 002 — fronteira de papel PO ↔ Analista.
- **Date**: 2026-07-14
- **Status**: active

### AD-016
- **Decision**: **Nenhum agente do Paperclip escreve de volta no Roadmap.** A marcação de sprint é **humana** (UI do Roadmap). O dedup do Analista vem da **própria memória** (histórico no Paperclip): antes de criar um PRD para um `codigo`, ele checa se já o tratou e pula se sim. Descartada a **escrita de sprint** pelo agente (`atribuir_sprint`, que o `Integracao-Paperclip.md`/ADR-031 do Roadmap previa como sinal de dedup).
- **Reason**: No momento da decomposição o agente **não sabe** a sprint real (decidida depois, no poker) — logo escrever sprint seria provisório e frágil. Manter a marcação humana + dedup por memória preserva a pureza read-only do pipeline e evita o reprocessamento da fila `semSprint`.
- **Trade-off**: O dedup depende de o Analista enxergar de forma confiável o próprio histórico no Paperclip; se essa memória falhar, há risco de reprocessar. A ser validado na construção do Increment 1.
- **Scope**: Feature 002 — dedup e fronteira de escrita no Roadmap.
- **Date**: 2026-07-14
- **Status**: active

---

## Aprendizados

Lições operacionais colhidas durante a construção manual do sistema no Paperclip. Registradas aqui porque
não há camada LESSONS automatizada neste projeto (sem código para verificar). Referenciam decisões acima.

- **O vetor de acesso real de um agente é `Bash` + CLI (`az`), não MCP.** Auditoria de configuração sozinha não pega esse caminho; só o teste funcional pega. → Reforça AD-002/AD-003 (contenção por credencial escopada) e AD-009 (contenção por run policy).
- **Instabilidade da sessão do Claude Code** (erros "Not logged in / resume session unavailable") faz o handoff *parecer* quebrado. Estabilizar o login é pré-requisito para testar comportamento. → Contexto para AD-006. **Ressalva (13/07/2026)**: a passada de consistência revelou que nem toda falha de handoff era da sessão — havia uma dessincronização real de instrução entre os dois lados do handoff (o `po/HEARTBEAT.md §5` e o `backlog-manager/HEARTBEAT.md` discordavam sobre onde ficavam a aprovação e o `decomposicao_json`). Resolvido pela **Forma B** (ver AD-010 e POBM-22): o PO copia o `decomposicao_json` inteiro no corpo da task de handoff; o BM lê do próprio corpo; e a existência da task de handoff é a autorização (o BM não navega entre tasks nem checa confirmation ACCEPTED). Lição: não atribuir toda instabilidade de comportamento à sessão sem antes cruzar as instruções dos dois lados do handoff.
- **O Paperclip não reacorda o PO de forma 100% confiável após a aprovação.** Quando trava, *Run Heartbeat* manual força o wake. O mecanismo nativo de automação do Paperclip (abas *Routines* e *Goals*) ainda não foi investigado como solução robusta. → Contexto para AD-006; ver Questões em aberto.
- **Identidade errada induz comportamento errado.** Um agente nomeado CEO puxa "ship over deliberate" e age sozinho. Corrigir a persona (`SOUL.md`) resolve. O PO herdou o nome de um PMO/CEO anterior e foi reescrito para o papel de PO. → Contexto para AD-001 e para o `SOUL.md` do PO.

---

## Questões em aberto

Pendências conhecidas no momento desta documentação. Não inventar respostas — cada item permanece aberto até ser resolvido.

- **Cerca #3 (gate de saída formal)** ainda não implementada além do confirmation atual (AD-004).
- **Investigar Routines/Goals do Paperclip** para tornar o wake pós-aprovação confiável (relacionado a AD-006 e ao aprendizado sobre o PO não reacordar).
- **`PAPERCLIP_API_KEY` no ambiente dos agentes**: dá poder de escrita interno no Paperclip, e os agentes a descobrem sozinhos. A estudar e conter.
- **Refinador-agente** (validador da Definition of Ready) decidido como próximo agente, ainda não construído. Deve ser ancorado em determinismo máximo (regras explícitas, saída estruturada). **Discrepância detectada (13/07/2026)**: o `agentes/po/TOOLS.md` já descreve o Refinador como validador **operacional** ("O Refinador audita a prontidão dos PBIs que você produz… um produz, o outro valida"), o que conflita com "ainda não construído". A confirmar com a Diana: o Refinador já existe, ou o texto no TOOLS.md é instrução prospectiva sobre um colaborador futuro? Enquanto não resolvido, tratar como não construído.
- **Conexão roadmap → Paperclip (feature 002)**: ver `features/002-conexao-roadmap-paperclip/spec.md`. **Desenho fechado em Specify (14/07/2026)**: pipeline de 4 agentes (Analista de Requisitos → Tech Lead sob demanda → PO → Backlog Manager), PULL via MCP (AD-012..016). A **barreira de rede localhost↔Vercel deixou de ser bloqueio** (AD-013): o cliente MCP é o runtime do agente, que chama a Vercel por HTTPS. **Pendências da construção**: (a) criar os agentes Analista e Tech Lead; (b) ajustar a Entrada do PO (produtor = Analista) e a DoR da skill (PO consome AS-IS/TO-BE, não produz o técnico — AD-015); (c) validar o papel read-only da key do Analista no Roadmap; (d) validar o dedup por memória do Analista; (e) o wake pós-ativação continua não confiável (herdado do AD-006).
- **Reference name do campo "História de Usuário" do PBI no Azure (não confirmado)**: o reference name interno do campo customizado não foi confirmado na sandbox `dianasandbox`. No `agentes/backlog-manager/TOOLS.md` o mapeamento de `historia_usuario` (PBI) está genérico ("campo de História de Usuário do PBI"). No teste E2E de 09/07/2026 a escrita funcionou, então ou o campo existe com um nome que o `az` aceitou, ou a história caiu na Description. **Pendência**: rodar `az boards work-item show` em um PBI criado, identificar onde a história foi parar e qual o reference name real do campo, e então fixar esse nome exato no `TOOLS.md` do Backlog Manager. **Não é bloqueante** (a escrita funciona) — é precisão de mapeamento. Relacionado a AD-005 e ao requisito POBM-12 da feature 001.
- **Sincronização Paperclip ↔ repo (pendente)**: os 7 arquivos de agente alterados nesta sessão estão versionados no git, mas o **Paperclip ainda roda as versões antigas** — falta colar o conteúdo novo de volta na UI. Alterados: `po/HEARTBEAT.md`, `po/AGENTS.md`, `po/SOUL.md`, `po/TOOLS.md`, `bm/HEARTBEAT.md`, `bm/AGENTS.md`, `bm/SOUL.md`. Sem mudança real: `po/DECOMPOSICAO.md`, `bm/TOOLS.md`. **Importante**: enquanto não sincronizado, o handoff Forma B (AD-010) não está ativo no runtime — o Paperclip ainda tem a versão anterior do handoff. **A repetir a cada edição futura destes arquivos** (o repo é a fonte de verdade; o Paperclip precisa ser atualizado a partir dele). **Nuance (AD-011)**: apenas o **método** (skill `skills/decomposicao-backlog/`) ganha o fluxo de import git + *install-update* (1 clique); os arquivos de **definição de agente** continuam sincronizados à mão por decisão consciente (a doc do Paperclip não expõe git-import para definição de agente).
- **Consolidar a duplicação do schema `decomposicao_json` (pendente)**: o schema aparece em 3+ lugares — `contratos/decomposicao-json-schema.md` (canônico do time), `skills/decomposicao-backlog/references/decomposicao-json-schema.md` (autossuficiência da skill) e embutido no `agentes/po/DECOMPOSICAO.md` (arquivo chapado a ser aposentado). Risco de divergência. Decidir a fonte canônica única e fazer os demais apontarem para ela quando a skill for validada e o `DECOMPOSICAO.md` colapsado.

---

## Handoff

- **Feature**: 001 (PO + Backlog Manager) documentada e versionada; método migrado para skill (AD-011). **002 (conexão roadmap→Paperclip)** com Specify fechada e **Increment 1 escrito no repo**.
- **Phase / Task**: 002 em **Execute (Increment 1)**. Criados os 4 arquivos do **Analista de Requisitos** (`agentes/analista-requisitos/`); ajustadas a Entrada do PO (produtor = Analista) e a DoR da skill (PO consome AS-IS/TO-BE — AD-015). Increment 2 (Tech Lead) ainda não construído.
- **Completed**: `.specs/` (STATE com AD-001..AD-016; spec 001 POBM-01..22; spec 002 CRP-01..13); `contratos/`; skill `skills/decomposicao-backlog/` (importada no Paperclip e anexada ao PO); agentes PO + Backlog Manager + **Analista de Requisitos** com conteúdo real; `agentes/po/DECOMPOSICAO.md` colapsado em tombstone.
- **In-progress** (file:line): none no repo. Increment 1 escrito; falta sincronizar no Paperclip.
- **Next step (próximos passos da Diana, fora do repo)**:
  1. **Sincronizar o Paperclip com o Increment 1**: criar o agente **Analista de Requisitos** no Paperclip e colar os 4 arquivos (`agentes/analista-requisitos/`); colar as edições da Entrada do PO (`po/AGENTS.md`, `po/HEARTBEAT.md`, `po/SOUL.md`); re-importar a skill Decomposição de Backlog por *install-update* (a DoR mudou). Aplicar as cercas do AD-009 no Analista.
  2. **Gerar a key `rmcp_` read-only** no Roadmap (`/agentes`, papel `gerente_projeto`), cadastrar como Secret no Paperclip e apontar o `.mcp.json` do runtime do Analista para `…/api/mcp` (passo a passo em `Roadmap/docs/Integracao-Paperclip.md`).
  3. **Teste funcional do Increment 1**: com uma iniciativa priorizada e sem sprint no Roadmap, ativar o Analista → PRD → PO decompõe → Diana aprova → handoff Forma B → Backlog Manager materializa. Validar o dedup (ativar o Analista 2x, PRD só na 1ª).
- **Blockers**: (1) Paperclip não sincronizado com o Increment 1. (2) Wake pós-ativação/aprovação ainda não confiável (herdado do AD-006). (A barreira de rede localhost↔Vercel deixou de ser blocker — AD-013.)
- **Uncommitted files**: os arquivos do Increment 1 (a commitar nesta sessão).
- **Branch**: main
- **Ao retomar**: se o teste funcional do Increment 1 rodar ponta a ponta, registrar como aprendizado; então abrir o **Increment 2** (agente Tech Lead) — Specify já esboçada (CRP-09..12).
