---
name: understand-easysb
description: Primer on EasySB, the 5-in-1 sing-box deployment panel for Linux. Use when the user asks what EasySB is, what it installs, which protocols or ports it serves, where it keeps its configuration, how its menu or command line is laid out, or how its subscription service works. Use this to explain EasySB; to install it on a server, call the deploy-easysb skill.
---

# Understand EasySB

EasySB is a single static Go binary, `easysb` (shortcut `sb`), that deploys and operates a five-protocol sing-box server on a Linux VPS. It runs a full-screen bubbletea TUI, issues TLS certificates in process, checks what the host IP can actually use, and serves one subscription URL per account. The sing-box core is compiled into the binary, so installing the panel installs the core.

Answer from `reference.md`. Read it before answering anything about protocols, ports, paths, services, the menu map, the command line, or the subscription service. Keep the answer short: name the fact, then the path or command that proves it.

## Where things are

- `reference.md` (beside this file): the fact sheet. Protocols and default ports, host requirements, runtime paths, systemd units, the subscription model, the main menu map, the command-line flags, and the facts that are easy to get wrong.

## What this skill does not do

- It does not run `easysb`, `sb`, or `install.sh`. It explains. For an install, call the Skill tool with "deploy-easysb".
- It does not guess. If a port, path, flag, or menu entry is not in `reference.md`, say so instead of inventing one.
