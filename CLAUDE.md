# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **documentation-only** repository — a VPS setup guide for Ubuntu 24.04 LTS. There are no build steps, test suites, or runnable code. All content lives in `docs/` as Markdown files.

Target stack: Python 3.11+, Docker, Nginx, MySQL 8, Telegram bots. Tools covered: UFW, CrowdSec, unattended-upgrades, Netdata.

## Document Structure

The `docs/` directory contains 12 sequentially numbered guides that must be followed in order:

1. `01-initial-setup.md` — system update, user creation, timezone, locale
2. `02-ssh-hardening.md` — ed25519 keys, disable root/password login
3. `03-ufw-firewall.md` — UFW rules (2222, 80, 443), rate limiting SSH
4. `04-crowdsec.md` — brute-force protection via CrowdSec (agent + iptables bouncer)
5. `05-docker-setup.md` — Docker + Docker Compose, daemon.json hardening
6. `06-nginx-reverse-proxy.md` — Nginx as reverse proxy, Certbot/HTTPS, security headers
7. `07-python-services.md` — deploy via Docker Compose (preferred) or systemd units; projects live in `/opt/projects/`
8. `08-unattended-upgrades.md` — automatic security updates
9. `09-monitoring.md` — Netdata, access restricted by IP or SSH tunnel
10. `10-checklist.md` — pre-deploy checklist covering all of the above
11. `11-vless-xray.md` — VLESS + Reality proxy via Xray (optional, circumvention)
12. `12-backup.md` — backup via Restic: incremental snapshots, Postgres pre-dump, systemd timer, retention policy

## Conventions

- **Language:** All document prose is written in Russian.
- **Audience:** A single operator setting up their own VPS from scratch as root, then switching to an unprivileged user (`myuser`).
- Each guide ends with a "Следующий шаг →" link pointing to the next document.
- `.env` files must never be committed; `chmod 600 .env` is the standard.
- The checklist (`10-checklist.md`) is the authoritative summary of what "done" looks like — keep it in sync when adding new sections.
