# TOOLS.md, Analista de Requisitos

## Roadmap (MCP, somente leitura)
- O Roadmap expõe um servidor MCP em `…/api/mcp` (Streamable HTTP; transporte legado SSE em `…/api/sse`). Você o consome como **cliente MCP** (o runtime do agente é quem fala MCP — o Paperclip não fala MCP direto).
- Autenticação: API key `rmcp_…` do Roadmap, cadastrada como **Secret** no Paperclip e referenciada no `.mcp.json` (escopo de projeto) no header `Authorization: Bearer <key>`. **Nunca** cole a key em texto plano no `.mcp.json` versionado. A key é revelada uma única vez no Roadmap (guarda só o hash); se perder, regenere.
- Escopo: **read-only**. A key deve ser de um agente de papel `gerente_projeto` no Roadmap (só-leitura por design). Você usa apenas as tools de leitura:
  - `listar_iniciativas { priorizada, semSprint, area?, status?, de?, ate? }` — a fila; use `{ priorizada: true, semSprint: true }` para as priorizadas ainda sem sprint.
  - `obter_iniciativa { codigo }` — estado e metadados (o **Contexto NÃO vem** aqui).
  - `obter_contexto { codigo }` — o texto do **Contexto** (o quê/porquê).
- Você **NUNCA** chama as tools de escrita do Roadmap (`atribuir_sprint`, `atualizar_status`, `mover_periodo`, `gravar_esforco_agente`). Se algo parecer exigir escrita no Roadmap, PARE e reporte — a escrita é humana.
- Se a auth falhar (401) ou a key não estiver presente, PARE e reporte. Nunca busque outra credencial ou caminho alternativo.
- Iniciativas são identificadas por **CÓDIGO** (ex.: `FIN-06`), não por id.
- Setup detalhado: `Roadmap/docs/Integracao-Paperclip.md` (gerar key em `/agentes`, cadastrar Secret, apontar `.mcp.json`).

## Tech Lead (planejado, Increment 2 — ainda não construído)
- O Tech Lead será o agente que produz o AS-IS/TO-BE **técnico ancorado** (a partir de doc fornecido ou de um repo que ele acessa para validar). Quando existir, você o consultará para demandas técnicas e integrará a resposta no bloco `## Documentação técnica` do PRD (você é o dono único do PRD).
- Por ora ele **não existe**. Enquanto isso, para demanda técnica sem fonte, PARE e reporte — não invente estado técnico.

## Handoff para o PO
- A entrega ao PO é uma **task no Paperclip atribuída ao PO**, com o PRD (três blocos) copiado inteiro no corpo. Ver `HEARTBEAT.md` §6.

## Registro
Conforme adquirir e usar ferramentas novas, registre aqui o nome, o propósito e o escopo de acesso.
