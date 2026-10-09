---
name: deploy-easysb
description: Interview the user for the details a deployment needs, then install and configure EasySB on their server. Use when the user wants to install or set up EasySB, asks to deploy EasySB on a VPS or a Linux server, wants a five-protocol sing-box server running with EasySB, or asks for an EasySB deployment with a specific domain, ports, or accounts.
---

# Deploy EasySB

Deployment has two halves. The agent can reach the host, install the package, and check the result. The panel is a full-screen TUI with no non-interactive mode, so the human creates the certificate, the nodes, and the accounts inside `sb`. This skill interviews the user, confirms one plan, runs the installing half, then hands off the panel half as exact menu paths.

## Process

### 1. Load the facts

Call the Skill tool with "understand-easysb", and read `reference.md` beside this skill. The interview and the handoff both depend on the protocol table, the default ports, and the host requirements.

### 2. Interview the user

Ask the questions in `reference.md` under **Interview**. Ask them in the user's own language, numbered so the user can answer by number, each with its default so the user can answer "defaults" and move on. Do not proceed until every required answer is known:

- how to reach the host,
- which protocols to enable,
- the domain that resolves to the host (for every protocol except VLESS + Reality),
- the ACME email,
- the accounts to create.

Never guess a domain, an email, a password, a port, or a username. Ask.

### 3. Confirm the plan

Turn the answers into the plan table in `reference.md` under **Plan**. Show the table and the exact commands the agent will run, then get an explicit yes before touching the host. If the user picked a protocol that needs a domain and no record resolves to the host, stop and resolve that first: the panel cannot issue a certificate without it.

Run this skill's side effects only after that yes.

### 4. Install

Work over SSH (or locally, if the user said the agent already runs on the server):

1. Confirm the OS and architecture against the host requirements in `understand-easysb`. Stop if the machine is not Debian 12/13 or Ubuntu 24.04, or not amd64/arm64.
2. Confirm the ports from the plan are reachable from outside: 80 for the challenge, each protocol port, the subscription port, and the hop range.
3. Install the panel:

```bash
curl -fsSL https://github.com/EasySBTeam/EasySB/releases/latest/download/install.sh | sudo bash
```

4. Verify the binary landed:

```bash
sb --version
```

### 5. Hand off the panel

The user runs `sb` on the server and drives the TUI. Walk them through the exact menu paths in `reference.md` under **Handoff**: domain issuance, one node per protocol, one account per username, and the subscription service. Give one path at a time and wait for the user to confirm each step, so nothing scrolls away.

### 6. Verify and report

Run the checks in `reference.md` under **Verify**: the version, both services, the config through `core check`, and the subscription URL. Report what the user must still do outside the panel, usually opening the ports in a cloud security group. Tell the user where the subscription URL is, and that the panel is the only management surface.

## Guardrails

- Confirm before anything irreversible, and never run the uninstall path as part of a deployment.
- Keep credentials out of the transcript. Read a subscription token from the panel rather than printing every account's secrets.
- If a step needs a value only the human can provide, ask for it; do not invent one to keep moving.
