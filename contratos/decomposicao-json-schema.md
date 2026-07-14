# Contrato: `decomposicao_json`

Contrato entre o **PO** (produtor) e o **Backlog Manager** (consumidor). O PO salva a árvore de decomposição
neste formato como documento `decomposicao_json`; o Backlog Manager lê e materializa a árvore no Azure Boards.

Relacionado: `.specs/features/001-sistema-po-backlog-manager/spec.md` (requisitos POBM-04, POBM-10..POBM-12).

## Schema

```json
{
  "iniciativa": "<id ou título da iniciativa>",
  "epic": {
    "titulo": "<título do Epic>",
    "descricao": "<objetivo estratégico do Epic>",
    "features": [
      {
        "titulo": "<título da Feature>",
        "descricao": "<contexto de negócio da Feature>",
        "pbis": [
          {
            "titulo": "<título do PBI>",
            "historia_usuario": "Como <ator>, quero <objetivo>, para <benefício>.",
            "description": "Contexto da demanda: <...>\n\nAS-IS: <...>\n\nTO-BE: <...>",
            "acceptance_criteria": "Critérios de Aceite:\n1. Dado <...>, quando <...>, então <...>\n\nCenários de Teste:\n- Cenário 1 — <...>",
            "tasks_dev": [
              "1 - <título da Task Dev>",
              "2 - <título da Task Dev>"
            ]
          }
        ]
      }
    ]
  }
}
```

## Regras do contrato

- É a **mesma árvore** do documento `decomposicao` (markdown), sem diferença de conteúdo.
- O **aninhamento define o parentesco**: `features` dentro do `epic`, `pbis` dentro da feature, `tasks_dev` dentro do pbi. O Backlog Manager cria de cima para baixo, passando o ID do pai como parent do filho.
- `description` carrega **Contexto, AS-IS e TO-BE juntos** (mapeia para o campo **Description** do Azure).
- `acceptance_criteria` carrega os **Critérios de Aceite e os Cenários de Teste juntos** (mapeia para o campo **Acceptance Criteria** do Azure).
- `tasks_dev` é uma **lista de títulos de Task**, cada um prefixado com número sequencial. Cada título vira um work item Task.
- O JSON **NÃO** inclui `Assigned To`, `Story Points`, `Area`, `Iteration` ou qualquer campo de atribuição ou estimativa (ver AD-005).

## Mapeamento de tipos JSON → Azure

| Nó no JSON | Tipo de work item no Azure |
| ---------- | -------------------------- |
| `epic`     | Epic |
| `feature`  | Feature |
| `pbi`      | Product Backlog Item |
| `task_dev` | Task |
