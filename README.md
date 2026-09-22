# Home Server Ansible Playbook

Rebuilds a Vaultwarden + Caddy + AdGuard Home + Paperless-ngx stack on
a fresh Ubuntu machine. Turns a set of manual SSH steps into a
repeatable, version-controlled setup.

> **Before you run this against your own machine:** replace the
> placeholder values in `inventory.ini` and `group_vars/all.yml`
> (IP address, username, Tailscale hostname, Paperless secrets) with
> your own. This repo ships with generic placeholders on purpose so
> it's safe to keep public.

## What this automates

- Power settings so the machine keeps running with the lid closed
- Docker + Docker Compose installation
- Freeing port 53 (disabling systemd-resolved's stub listener) so
  AdGuard Home can bind to it
- Deploying `docker-compose.yml` and `Caddyfile` from templates
- Starting the full container stack (Vaultwarden, Caddy, AdGuard,
  Paperless-ngx)
- Setting up nightly backup scripts + cron jobs for Vaultwarden and
  Paperless

## Usage

```bash
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass --ask-become-pass 
```

Adjust variables (hostnames, ports, retention policy) in
`group_vars/all.yml` before running.

## Secrets

Paperless-ngx requires a database password and a secret key, set as
`paperless_db_password` and `paperless_secret_key` in
`group_vars/all.yml`. These should be encrypted with `ansible-vault`
rather than committed in plaintext:

```bash
ansible-vault encrypt_string 'your-password' --name 'paperless_db_password'
```

Paste the resulting block into `group_vars/all.yml` in place of a
plain value, then pass `--ask-vault-pass` when running the playbook.

## Accessing services

Vaultwarden, Paperless, and AdGuard's web UI are all served through
Caddy under one Tailscale hostname, split by subpath:

- Vaultwarden: `https://{{ tailscale_hostname }}/vaultwarden`
- Paperless: `https://{{ tailscale_hostname }}/paperless`

AdGuard's web UI is exposed on its own port
(`{{ adguard_web_port }}`) rather than a subpath.

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
    ├── backup-vaultwarden.sh.j2
    └── backup-paperless.sh.j2
```