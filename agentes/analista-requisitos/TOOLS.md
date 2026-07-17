# TOOLS.md, Analista de Requisitos

## Roadmap (MCP, somente leitura)
- O Roadmap expõe um servidor MCP em `…/api/mcp` (Streamable HTTP; transporte legado SSE em `…/api/sse`). Você o consome como **cliente MCP** (o runtime do agente é quem fala MCP — o Paperclip não fala MCP direto).
- Autenticação e setup são **da Diana, não seus**: ela gera a key `rmcp_…` no Roadmap, cadastra como Secret no Paperclip e monta o `.mcp.json`. Você **não configura o MCP** — apenas **usa** as tools do Roadmap que já aparecem no seu ambiente.
- Escopo: **read-only**. A key deve ser de um agente de papel `gerente_projeto` no Roadmap (só-leitura por design). Você usa apenas as tools de leitura:
  - `listar_iniciativas { priorizada, semSprint, area?, status?, de?, ate? }` — a fila; use `{ priorizada: true, semSprint: true }` para as priorizadas ainda sem sprint.
  - `obter_iniciativa { codigo }` — estado e metadados (o **Contexto NÃO vem** aqui).
  - `obter_contexto { codigo }` — o texto do **Contexto** (o quê/porquê).
- Você **NUNCA** chama as tools de escrita do Roadmap (`atribuir_sprint`, `atualizar_status`, `mover_periodo`, `gravar_esforco_agente`). Se algo parecer exigir escrita no Roadmap, PARE e reporte — a escrita é humana.
- **Condição de PARADA (dura).** Se as tools do Roadmap (`listar_iniciativas`, `obter_contexto`) NÃO aparecerem no seu ambiente, ou a chamada falhar (401 / inacessível): PARE e reporte à Diana que **o MCP do Roadmap não está disponível/configurado**. É ela quem configura. Você NÃO procura key no workspace ou no ambiente, NÃO tenta a API do Paperclip (`127.0.0.1:3100` ou qualquer outra), NÃO procura um servidor do Roadmap rodando local, NÃO configura o MCP você mesmo. **Nenhum caminho alternativo** — a ausência da tool é o fim da linha, não um problema a resolver.
- Iniciativas são identificadas por **CÓDIGO** (ex.: `FIN-06`), não por id.
- Setup detalhado (para a Diana, não para você): `Roadmap/docs/Integracao-Paperclip.md`.

## Tech Lead (planejado, Increment 2 — ainda não construído)
- O Tech Lead será o agente que produz o AS-IS/TO-BE **técnico ancorado** (a partir de doc fornecido ou de um repo que ele acessa para validar). Quando existir, você o consultará para demandas técnicas e integrará a resposta no bloco `## Documentação técnica` do PRD (você é o dono único do PRD).
- Por ora ele **não existe**. Enquanto isso, para demanda técnica sem fonte, PARE e reporte — não invente estado técnico.

## Handoff para o PO
- A entrega ao PO é uma **task no Paperclip atribuída ao PO**, com o PRD (três blocos) copiado inteiro no corpo. Ver `HEARTBEAT.md` §6.

## Registro
Conforme adquirir e usar ferramentas novas, registre aqui o nome, o propósito e o escopo de acesso.
