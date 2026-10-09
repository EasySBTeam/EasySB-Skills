# EasySB fact sheet

The source of truth for `understand-easysb`. Everything here comes from the EasySB repository (`README.md`, `AGENTS.md`, `docs/`) and the packaged installer.

## What it is

- One static Go binary, `easysb`, with the shortcut `sb`. Module `github.com/EasySBTeam/EasySB`, Go 1.27.1.
- Deploys and operates a five-protocol sing-box server on a Linux VPS, through a full-screen bubbletea TUI.
- sing-box is a `go.mod` requirement, compiled into the binary. The node is `easysb core run -c /etc/sing-box/config.json`. There is no separate core to download, replace, or switch.
- Certificates are issued in process through `go-acme/lego` with the HTTP-01 standalone challenge. No `acme.sh`, no `socat`, nothing downloaded.
- A built-in subscription service serves one URL per account, negotiated from the client's User-Agent.
- Runtime management is the panel, or the headless `sb --provision FILE` mode that deploys a whole host from one JSON manifest. There is no web panel.

## Protocols and default ports

One node is one protocol inbound with its own port and its own parameters.

| Protocol | Transport | Default port | Needs a domain and a certificate |
| :--- | :--- | :--- | :--- |
| AnyTLS | TCP + TLS | 8000 | Yes |
| Hysteria2 | QUIC / UDP | 8001 | Yes. Port hopping defaults to `2080:3000` |
| TUIC v5 | QUIC / UDP | 8002 | Yes |
| VLESS + Vision + Reality | TCP | 8003 | No. Borrows `apple.com` as the SNI by default |
| VMess + WebSocket + TLS | WS over TLS | 8004 | Yes |

Every protocol except VLESS + Reality needs a domain that already resolves to the host, plus a valid certificate. Enter takes the protocol default port, `r` picks a random port, a number sets it manually. A port already used by another enabled node or by the subscription service is rejected.

## Host requirements

- A Linux VPS. Supported releases: Debian 12 and 13, Ubuntu 24.04. Supported architectures: amd64 and arm64. The BBR kernels cover the same set.
- root, or a user with sudo.
- A domain whose A/AAAA record resolves to the host, for every protocol except Reality.
- An email address for the Let's Encrypt (ACME) account.
- Open ports: 80 for the HTTP-01 challenge, each enabled protocol's port, the subscription port (default 8443), and the Hysteria2 UDP hop range when port hopping is on.

## Runtime paths

| Path | Contents |
| :--- | :--- |
| `/usr/bin/easysb` | The panel, with the core compiled in |
| `/usr/bin/sb` | Shortcut for `easysb` |
| `/etc/sing-box/config.json` | The rendered server config |
| `/etc/sing-box/easysb.conf` | Panel state |
| `/etc/sing-box/easysb-nodes.json` | Node store: the only source of what the host serves |
| `/etc/sing-box/easysb-users.json` | Account store: the only source of credentials |
| `/etc/apt/sources.list.d/easysb.list` | The apt source |
| `/usr/share/keyrings/easysb-archive-keyring.gpg` | The signing key |

## systemd units

| Unit | Job |
| :--- | :--- |
| `sing-box.service` | The node: `easysb core run -c /etc/sing-box/config.json` |
| `easysb.service` | The subscription service: `easysb --serve` |
| `easysb-firewall.service` | Restores port-hopping rules at boot. Created only when port hopping is on |
| renewal timer | Renews certificates and reloads the services |

A fresh package install deliberately does not enable or start either service: there is no node configuration yet. Configuring a node makes the panel enable and start them.

## Subscription model

- One URL per account: `/sub/<token>`, served by the built-in service on `SUB_SERVE_PORT` (default 8443).
- The response is negotiated from the User-Agent, so one URL works everywhere:

| Client | Response |
| :--- | :--- |
| sing-box (SFM / SFA / SFI) | JSON profile |
| mihomo / Clash Meta / luci-app-nikki | Complete YAML profile |
| v2rayN / passwall / passwall2 / homeproxy | Base64 share-link document |

`?client=singbox\|mihomo\|v2ray` overrides the detection. The token in the URL is the access secret; rotate it or disable the account to revoke one person without touching anyone else. Every response carries `Subscription-Userinfo` for remaining traffic and days. A disabled, expired, or over-quota account gets `403` with a plain-text reason. The service terminates TLS itself when a real certificate is installed for the domain; otherwise it serves plain HTTP and the panel warns, because a subscription carries credentials. Traffic is sampled every `SUB_SYNC_SECONDS` (default 300).

## Main menu map

```text
Toolbox                Unlock checks, network, IP and ports, hardware and performance
Node management        Add, edit, enable / disable, set the port and protocol parameters, delete
Domain management      Issue, renew now, renewal timer, list, switch active, remove
Subscription           One account's URL, QR code and share links; service install / restart / status
Accounts               List, create, rename, remark, quota, expiry, node selection, enable / disable, usage reset, token rotation, delete
Service management     Start, stop, restart, status, enable / disable, port-hopping rules
System info            Runtime, skin / palette / markers / language, terminal and host details
BBR                    Status, enable BBR, install the standard or Max BBRv3 kernel, remove, clear
Update version         Pull the latest EasySB release
Uninstall script       Remove EasySB completely
```

Keys: `Q` leaves the panel from any page, `Esc` goes back, `Enter` only enters or confirms.

## Command line

| Flag | Job |
| :--- | :--- |
| `--language C\|E` | Preset the UI language, then open the menu |
| `--icons symbols\|ascii` | Marker set |
| `--theme auto\|dark\|light` | Override terminal background detection |
| `--skin jade\|aurora\|ember\|graphite` | Interface skin |
| `--apply-firewall` | Restore port-hopping rules only, used by the boot unit |
| `--renew-certs` | Renew every certificate, used by the renewal timer |
| `--install-renew-timer` / `--remove-renew-timer` | Manage the renewal timer |
| `--render --width N --height N` | Render the dashboard once and exit |
| `--serve` | Run the subscription service and accounting loop |
| `--provision FILE` | Deploy from a JSON manifest and exit without the menu; `-` reads the manifest from stdin |
| `--print-unit node\|sub` | Print a service unit body to stdout |
| `--version` | Print the version and build hash |

The core subcommand is `easysb core run|check|version [-c <config>]`. `core check` validates a configuration with the same engine the node uses.

## Headless provisioning

`sb --provision FILE` reads one JSON manifest and performs the whole deployment: it issues the certificate, writes the node store and the account store, renders the core config, writes and starts the service units, and prints each account's subscription URL. The manifest is a desired state rather than a script, so re-running it reuses the nodes it already finds and keeps each account's token and credentials: a subscription URL a client already imported does not change. Fields: `domain`, `email`, `server_ip`, `sub_port`, `nodes[]` (`protocol`, `name`, `port`, `sni`, `hop_range`) and `accounts[]` (`name`, `password`, `uuid`, `quota_gb`, `expire_days`, `nodes[]`). An omitted `nodes` list means all five protocols at their default ports. Unknown keys are rejected. The `deploy-easysb` skill builds this manifest, so call it rather than writing one by hand.

## Facts that are easy to get wrong

- A host can be deployed without the panel through `sb --provision FILE`; the panel is the interactive surface for changes after that.
- Every protocol except Reality needs a domain that resolves to the host and a valid certificate.
- A fresh install leaves the services stopped on purpose; configuring a node starts them.
- Debian 12/13 and Ubuntu 24.04 only, amd64 and arm64 only.
- The rule sets used by the subscription templates come from `EasySBTeam/Proxy-Rules-Data`.
