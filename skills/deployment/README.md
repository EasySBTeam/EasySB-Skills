# Deployment

Skills for getting EasySB onto a server and for explaining what it is once it is there.

Both are **model-invoked**: your agent reaches for them when your request fits, and you can also call them by name.

- **[deploy-easysb](./deploy-easysb/SKILL.md)**: Interview the user for the details a deployment needs, then install and configure EasySB on their server.
- **[understand-easysb](./understand-easysb/SKILL.md)**: Primer on EasySB: protocols, ports, paths, services, menu map and command line.

The two fit together: `deploy-easysb` calls `understand-easysb` for the facts before it interviews, so the questions and the handoff steps stay correct as EasySB changes.
