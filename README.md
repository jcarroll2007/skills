# Skills

A collection of [Claude](https://claude.com/claude-code) skills for authoring
requirements documents. Each skill is **independent** — install only the ones you
want.

## Installation

Install with the [`skills`](https://www.npmjs.com/package/skills) CLI:

```bash
npx skills@latest add jcarroll2007/skills
```

The installer lets you **select which skills to add** and which coding agents to
install them on, so every skill in this repo is independently installable.

> Update `jcarroll2007/skills` to match your GitHub `owner/repo` if you fork or
> rename this repository.

## Skills

| Skill | Description |
| ----- | ----------- |
| [`prd`](./skills/prd) | Author a **Product Requirements Document** — problem, users, solution, success metrics, and scope. |
| [`emd`](./skills/emd) | Author an **Engineering Milestone Document** — the engineering end state and how the journey phases into independently shippable milestones. |
| [`erd`](./skills/erd) | Author an **Engineering Requirements Document** — testable functional/non-functional requirements, constraints, and acceptance criteria. |
| [`rfc`](./skills/rfc) | Author an **engineering design doc / RFC** — the chosen approach, why it beats the alternatives, and what could go wrong. |
| [`adr`](./skills/adr) | Author an **Architecture Decision Record** — a durable, immutable record of one decision, its context, and its consequences. |

## Repository structure

```
.
├── .claude-plugin/
│   └── plugin.json        # Lists each skill so it can be installed independently
├── skills/
│   ├── prd/                # each skill: SKILL.md + TEMPLATE.md + EXAMPLE.md
│   ├── emd/
│   ├── erd/
│   ├── rfc/
│   └── adr/
├── package.json
└── README.md
```

To add a new skill: create a folder under `skills/` containing a `SKILL.md`, then
add its path to the `skills` array in `.claude-plugin/plugin.json`.

## License

[MIT](./LICENSE)
