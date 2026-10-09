# EasySB deployment worksheet

The interview, the plan template, the panel handoff, and the verification for `deploy-easysb`. Facts about EasySB itself live in `understand-easysb`.

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

### Group 4: the accounts

8. Which usernames should I plan? For each one, the traffic quota, the expiry date, and which nodes they may use. Default: one account per username, no quota, no expiry, all nodes.
9. Install the subscription service? Default: yes, since it is how clients import the account.

### Group 5: the extras

10. Enable BBR? Default: yes, with the `fq` queue discipline.
11. Panel language: Chinese (`C`) or English (`E`)? Default: Chinese.
12. Is there a cloud security group or a local firewall (ufw / firewalld) in front of the server? It decides which ports the user must open after the install.

## Plan

Fill this in and show it before touching the host.

| Item | Value |
| :--- | :--- |
| Host | `user@host:port` |
| OS / arch | e.g. Debian 13 / amd64 |
| Domain | e.g. `node.example.com`, resolves: yes / no |
| ACME email | e.g. `admin@example.com` |
| Protocols and ports | e.g. AnyTLS 8000, Hysteria2 8001, TUIC 8002, VLESS + Reality 8003 |
| Accounts | e.g. `alice`, all nodes, 100 GB, expires 2027-01-01 |
| Subscription | enabled, port 8443 |
| BBR | on, `fq` |
| Language | Chinese |
| Ports to open | 80, each protocol port, 8443, UDP hop range |

## Handoff

The human runs `sb` on the server. Walk one path at a time and wait for confirmation before the next.

1. **Domain management -> Issue.** Enter the domain and the ACME email from the plan. Wait for the DNS preflight to pass and the certificate to issue. Skip this step when the plan is Reality-only.
2. **Node management -> Add.** One node per protocol in the plan. `Enter` takes the default port, or type the plan's port. For Reality set the SNI and keypair; for Hysteria2 set the hop range. Enable each node.
3. **Accounts -> Create.** One account per username. Set its quota, expiry, and node selection from the plan.
4. **Subscription -> install** the service, then pick each account and copy its `/sub/<token>` URL, QR code, or share links.
5. **Service management -> start** and enable on boot.
6. **BBR -> enable** with the planned queue discipline, if the plan turns it on.

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
- where the subscription URL is, and that the panel on the server is the only management surface.
