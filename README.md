# arch-data

Architecture-as-code data repository. It holds the enterprise architecture model —
components, solutions, diagrams and generated documentation — as plain YAML, JSON,
Markdown and draw.io files, so the whole model is diffable, reviewable and versioned
in Git instead of living inside a modelling tool.

The repository is data only. There is no application code here; an architecture tool
reads and writes these files, and every change it makes lands as a normal commit.

## Layout

| Path | Contents |
| --- | --- |
| `components/` | One YAML file per architecture building block (service, database, gateway, external system, …). |
| `solutions/` | Solution designs that compose components into a delivery: goal, delivered capabilities, members, flows, processes. |
| `dsd/<solution-id>/` | Generated Detailed Solution Descriptions (Markdown with YAML front matter), one file per run, named by timestamp. |
| `agents/` | Definitions of the LLM agents that write the DSDs — system prompt, role, temperature, version, accumulated lessons. |
| `diagrams/` | draw.io / mxGraph diagrams (`.drawio`). |
| `confluence-links/` | Mapping between a component and the Confluence page it is published to, plus last-sync state. |
| `config.yaml` | UI configuration — which blocks are shown in the tool's overview, technical and business views. |

## Component files

Each file in `components/` is keyed by its `id`, which matches the filename and is
what everything else refers to.

```yaml
id: api-gateway
name: API Gateway
type: gateway            # microservice, database, service, frontend, external,
                         # gateway, storage, queue, platform, module, library,
                         # data-pipeline, context, cache, boundary, batch-job, application
status: production       # production | draft
owner: platform-team
tags: [core, networking, security]

description:
  oneliner: Central entry point for all API requests
  technical: …           # how it is built and operated
  business: …            # why it exists, in business terms

risks: [ … ]             # free-text risk statements

capabilities:            # business capabilities the component supports
  - name: Customer Management
    role: indirect       # direct | indirect

processes:               # business processes it participates in
  - name: process A
    role: participant
    activity: supports step A

rules:                   # formulas, rules and constraints the component enforces
  - name: Per-tier rate limit
    kind: formula        # formula | rule | constraint
    summary: …
    formula: allowed_rps = base_rps * tier_multiplier

nfr:                     # non-functional requirements
  availability: 99.99%
  max_latency: 200ms
  throughput: 12r/s
  rto: '4'
  rpo: 10 mins
  data_classification: public
  scaling: vertical

diagram:                 # rendering hints
  color: '#3B82F6'
  shape: hexagon

links:                   # relationships to other components
  - target: identity-service
    role: calls          # calls | serves | reads-from | writes-to | contains | part-of
    protocol: rest       # rest | db | async | file
    description: JWT token validation

schema_version: 2
```

### Schema versions

`schema_version: 2` is the current shape. Older files may still carry the earlier
`interfaces:`, `relationships:` and `dependencies:` blocks instead of `links:`, and may
omit `schema_version` entirely. Both are readable; new and updated components should use
`links:`.

## Solution files

A solution in `solutions/` describes a change to the landscape rather than a single
building block:

- `goal` and `delivers.capabilities` — what the solution is for.
- `members` — the components involved, each with a `disposition` of `new`, `extend` or
  `reuse`, and the role it plays.
- `flows` — the connections between members, each marked `existing` or `proposed`, so a
  target-state diagram can be drawn straight from the file.
- `processes` — actors and steps for the business processes the solution supports.

## Generated documentation

`dsd/` holds Detailed Solution Descriptions produced from a solution file by the agents
in `agents/`. The front matter records which solution and which agent versions produced
the document, how many iterations it took, and any reviewer feedback left on it. The
files are outputs — regenerate them rather than editing them by hand, and let feedback
flow back into the agent definitions.

## Conventions

- One entity per file; the filename is the `id`.
- Component and solution ids are lowercase kebab-case and stable — other files reference
  them by id, so renaming one means updating every reference.
- Keep diffs small and readable: change the model, not the formatting.
- Descriptions are prose meant for humans. State facts plainly and skip marketing language.

## License

[MIT](LICENSE)
