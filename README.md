# Home Server Ansible Playbook

Rebuilds a Vaultwarden + Caddy + AdGuard Home stack on a fresh Ubuntu
machine. Turns a set of manual SSH steps into a repeatable,
version-controlled setup.

> **Before you run this against your own machine:** replace the
> placeholder values in `inventory.ini` and `group_vars/all.yml`
> (IP address, username, Tailscale hostname) with your own. This repo
> ships with generic placeholders on purpose so it's safe to keep
> public.

## What this automates

- Power settings so the machine keeps running with the lid closed
- Docker + Docker Compose installation
- Freeing port 53 (disabling systemd-resolved's stub listener) so
  AdGuard Home can bind to it
- Deploying `docker-compose.yml` and `Caddyfile` from templates
- Starting the full container stack (Vaultwarden, Caddy, AdGuard)
- Setting up the nightly backup script + cron job

## Usage

```bash
ansible-playbook -i inventory.ini playbook.yml --ask-become-pass
```

Adjust variables (hostnames, ports, retention policy) in
`group_vars/all.yml` before running.

## File structure

```
.
├── inventory.ini              # target host(s) — placeholder values
├── playbook.yml                # main playbook
├── group_vars/
│   └── all.yml                 # configurable variables — placeholder values
└── templates/
    ├── docker-compose.yml.j2
    ├── Caddyfile.j2
    └── backup-vaultwarden.sh.j2
```

## Requires

- `community.docker` collection:
  ```bash
  ansible-galaxy collection install community.docker
  ```
- SSH access + sudo privileges on the target host
