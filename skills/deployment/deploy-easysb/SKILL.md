---
name: deploy-easysb
description: Interview the user for the details a deployment needs, then install and configure EasySB on their server. Use when the user wants to install or set up EasySB, asks to deploy EasySB on a VPS or a Linux server, wants a five-protocol sing-box server running with EasySB, or asks for an EasySB deployment with a specific domain, ports, or accounts.
---

# Deploy EasySB

EasySB ships a headless deployment mode, `sb --provision`. It turns one JSON manifest of the desired state into the certificate, the nodes, the accounts, the rendered core config, and the running services, then prints each account's subscription URL. This skill interviews the user, turns the answers into that manifest, confirms one plan, installs the package, and runs the manifest over SSH. The user never has to drive the panel.

## Process

### 1. Load the facts

Call the Skill tool with "understand-easysb", and read `reference.md` beside this skill. The interview and the manifest both depend on the protocol table, the default ports, and the host requirements.

### 2. Interview the user

Ask the questions in `reference.md` under **Interview**. Ask them in the user's own language, numbered so the user can answer by number, each with its default so the user can answer "defaults" and move on. Do not proceed until every required answer is known:

- how to reach the host,
- which protocols to enable,
- the domain that resolves to the host (for every protocol except VLESS + Reality),
- the ACME email,
- the accounts to create,
- the subscription port.

Never guess a domain, an email, a password, a port, or a username. Ask.

### 3. Build the manifest

Fill the template in `reference.md` under **Manifest**, one node per protocol in the plan and one entry per account, using the user's own ports, usernames, quotas and expiry dates. Name each protocol explicitly whenever the plan is not all five at their default ports.

### 4. Confirm the plan

Show the plan table in `reference.md` under **Plan**, the finished manifest, and the exact commands the agent will run. Get an explicit yes before touching the host. If the user picked a protocol that needs a domain and no record resolves to the host, stop and resolve that first: provisioning cannot issue a certificate without it.

Run this skill's side effects only after that yes.

### 5. Install

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

### 6. Provision

Run the manifest on the server. Pipe it over SSH so no file is left behind:

```bash
ssh user@host:port 'sudo sb --provision -' <<'JSON'
<the confirmed manifest>
JSON
```

The command prints one line per step and ends with the nodes it deployed and every account's `https://<domain>:<sub_port>/sub/<token>` URL. Read those URLs from the output; keep raw passwords and UUIDs out of the transcript.

`--provision` is idempotent: re-running the same manifest reuses the nodes it already finds and keeps each account's token, so a subscription URL a client already imported does not change. A failed run stops before it renders a broken config, so fix the reason it printed and run it again.

### 7. Verify and report

Run the checks in `reference.md` under **Verify**: the version, both services, the config through `core check`, and the subscription URL. Report what the user must still do outside the panel, usually opening the ports in a cloud security group. Tell the user where the subscription URL is, and that the panel on the server remains the surface for later changes.

## Guardrails

- Confirm before anything irreversible, and never run the uninstall path as part of a deployment.
- Keep credentials out of the transcript. Report subscription URLs, not raw passwords or UUIDs.
- If a step needs a value only the human can provide, ask for it; do not invent one to keep moving.
