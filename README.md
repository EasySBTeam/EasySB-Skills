# EasySB Skills

Agent skills that let a coding agent understand [EasySB](https://github.com/EasySBTeam/EasySB) and deploy it on a server for you.

EasySB is a 5-in-1 sing-box deployment tool for a Linux VPS: one static Go binary (`easysb`, shortcut `sb`) that deploys and operates a five-protocol sing-box server, issues TLS certificates in process, and serves one subscription URL per account. The panel is a full-screen TUI with no non-interactive mode, so a skill cannot click through it for you.

What a skill can do is know the tool cold, ask every question a deployment raises, run the installing half over SSH, and hand you the panel half as exact menu paths. That is this repo.

## Install

Pick one route per agent. The plugin routes update themselves; the editable route writes files you own.

### Claude Code

```bash
claude plugin marketplace add EasySBTeam/EasySB-Skills
claude plugin install easysb-skills@easysb
```

### Codex

```bash
codex plugin marketplace add EasySBTeam/EasySB-Skills
codex plugin add easysb-skills@easysb
```

### GitHub Copilot (CLI and VS Code)

```bash
copilot plugin marketplace add EasySBTeam/EasySB-Skills
copilot plugin install easysb-skills@easysb
```

In VS Code, run **Chat: Install Plugin From Source** and enter `https://github.com/EasySBTeam/EasySB-Skills`.

### Gemini CLI

```bash
gemini skills install https://github.com/EasySBTeam/EasySB-Skills.git --path skills/deployment
```

Re-run the command to update.

### Any other agent, or editable files

```bash
npx skills@latest add EasySBTeam/EasySB-Skills
```

To update, run `npx skills@latest update`, and re-run `add` to pick up new skills.

## Skills

Every skill is model-invoked: your agent reaches for it when your request fits, and you can also call it by name.

| Skill | Job |
| :--- | :--- |
| [deploy-easysb](./skills/deployment/deploy-easysb/SKILL.md) | Interview you for the details a deployment needs, confirm one plan, install EasySB over SSH, then walk you through the panel steps only you can perform. |
| [understand-easysb](./skills/deployment/understand-easysb/SKILL.md) | Answer questions about what EasySB is, which protocols and ports it serves, where it stores configuration, and how its subscription service works. |

## License

GPL-3.0. See [LICENSE](./LICENSE).
