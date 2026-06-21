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
| [`erd`](./skills/erd) | Author an **Engineering Requirements Document** — architecture, data models, interfaces, dependencies, and rollout. |

## Repository structure

```
.
├── .claude-plugin/
│   └── plugin.json        # Lists each skill so it can be installed independently
├── skills/
│   ├── prd/
│   │   └── SKILL.md
│   └── erd/
│       └── SKILL.md
├── package.json
└── README.md
```

To add a new skill: create a folder under `skills/` containing a `SKILL.md`, then
add its path to the `skills` array in `.claude-plugin/plugin.json`.

## License

[MIT](./LICENSE)
