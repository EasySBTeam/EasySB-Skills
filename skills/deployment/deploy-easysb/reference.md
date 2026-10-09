# EasySB deployment worksheet

The interview, the manifest template, the plan table, and the verification for `deploy-easysb`. Facts about EasySB itself live in `understand-easysb`.

## Interview

Ask these in the user's language, numbered, each with its default. Stop at the end of each group and wait for answers.

### Group 1: the host

1. How should I reach the server? Give the SSH host (IP or name), the port, and the user. Say "this machine" if I already run on it.
2. What OS and CPU does it run? I need Debian 12 or 13, or Ubuntu 24.04, on amd64 or arm64.
3. What is the server's public IP or hostname?

### Group 2: the domain and certificate

4. Which domain should EasySB use, and does its A/AAAA record already point at this server? Skip if you only want VLESS + Reality.
5. Which email should Let's Encrypt use for expiry notices? Skip with question 4.

### Group 3: the nodes

6. Which protocols do you want? AnyTLS, Hysteria2, TUIC v5, VLESS + Vision + Reality, VMess + WebSocket + TLS. Default: all five.
7. Keep the default ports (8000 to 8004) or set your own? Turn Hysteria2 port hopping on, and keep the default `2080:3000` range? Default: default ports, hopping on.
8. Which port should the subscription service listen on? Default: 8443.

### Group 4: the accounts

9. Which usernames should I plan? For each one, the traffic quota, the expiry in days, and which nodes they may use. Default: one account per username, no quota, no expiry, all nodes.

### Group 5: the extras

10. Enable BBR? Default: yes, with the `fq` queue discipline. This is a panel step afterward, not part of the manifest.
11. Is there a cloud security group or a local firewall (ufw / firewalld) in front of the server? It decides which ports the user must open after the install.

## Manifest

Turn the answers into this document. Every field is optional: an omitted `nodes` list means all five protocols at their default ports; an omitted node `port` uses the protocol default; an omitted node `name` uses the protocol label.

```json
{
  "domain": "node.example.com",
  "email": "admin@example.com",
  "server_ip": "203.0.113.10",
  "sub_port": 8443,
  "nodes": [
    { "protocol": "anytls" },
    { "protocol": "hysteria2", "port": 8001, "hop_range": "2080:3000" },
    { "protocol": "tuic" },
    { "protocol": "vless-reality" },
    { "protocol": "vmess-ws-tls" }
  ],
  "accounts": [
    { "name": "alice", "quota_gb": 100, "expire_days": 365 },
    { "name": "bob", "nodes": ["vless-reality"] }
  ]
}
```

Field notes:

- `domain` and `email` are required as soon as any protocol other than VLESS + Reality is in the list. Drop both for a Reality-only host.
- `nodes[].protocol` is the key, not the label: `anytls`, `hysteria2`, `tuic`, `vless-reality`, `vmess-ws-tls`.
- `nodes[].port` must be 1-65535, unique, and different from `sub_port`.
- `nodes[].sni` overrides the Reality SNI (default `apple.com`); `nodes[].hop_range` overrides the Hysteria2 hop range (default `2080:3000`).
- `accounts[].nodes` matches a node by protocol key or by node name, and defaults to every node.
- `accounts[].quota_gb` 0 means unlimited; `accounts[].expire_days` 0 means no expiry.
- `accounts[].password` and `accounts[].uuid` set the credential the protocol uses; leave them out to let provisioning generate one.
- Unknown keys are rejected, so a typo fails before anything is written.

## Plan

Fill this in and show it before touching the host.

| Item | Value |
| :--- | :--- |
| Host | `user@host:port` |
| OS / arch | e.g. Debian 13 / amd64 |
| Domain | e.g. `node.example.com`, resolves: yes / no |
| ACME email | e.g. `admin@example.com` |
| Protocols and ports | e.g. AnyTLS 8000, Hysteria2 8001, TUIC 8002, VLESS + Reality 8003 |
| Accounts | e.g. `alice`, all nodes, 100 GB, 365 days |
| Subscription | port 8443 |
| BBR | on, `fq` (panel step after provisioning) |
| Ports to open | 80, each protocol port, 8443, UDP hop range |

## Provision

Pipe the confirmed manifest into the mode over SSH, so nothing is written to the server but the stores themselves:

```bash
ssh user@host:port 'sudo sb --provision -' <<'JSON'
{
  "domain": "node.example.com",
  "email": "admin@example.com",
  "accounts": [{ "name": "alice" }]
}
JSON
```

The run stops at the first reason it cannot continue and prints it, so a bad port, an unknown protocol, or a domain that is not yet resolvable is caught before the core is touched. On success it prints the deployed nodes and each account's `https://<domain>:<sub_port>/sub/<token>` URL.

## Verify

From the server:

```bash
sb --version
systemctl status sing-box
systemctl status easysb
easysb core check -c /etc/sing-box/config.json
```

From outside, once the ports are open:

```bash
curl -fsS https://<domain>:8443/sub/<token>
```

The response is a client profile. Use `http://` when no certificate is installed for the domain.

Close out by telling the user:

- which ports still need opening in the cloud security group,
- where the subscription URL is, and that the panel on the server is where later changes are made.

## Panel extras

These are outside the manifest, so hand them to the user as menu paths only if they asked for them:

1. **BBR -> enable** with the planned queue discipline.
2. **Subscription -> service** to restart or inspect the subscription service.
3. **System info** to set the panel language, skin or markers.
