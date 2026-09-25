# Home Server Ansible Playbook

Rebuilds a Vaultwarden + Caddy + AdGuard Home + OpenCloud + Paperless-ngx + Grafana + Prometheus + cAdvisor + Beszel + uptime Kuma stack on a fresh Ubuntu machine. Turns a set of manual SSH steps into a repeatable, version-controlled setup.

Before you run this against your own machine: replace the placeholder values in `inventory.ini` and `group_vars/place-holder.yml` (IP address, username, Tailscale hostname, Paperless secrets, Beszel token/key) with your own. This repo ships with generic placeholders on purpose so it's safe to keep public.

## What this automates

* Power settings so the machine keeps running with the lid closed
* Docker + Docker Compose installation
* Freeing port 53 (disabling systemd-resolved's stub listener) so AdGuard Home can bind to it
* Deploying `docker-compose.yml` and `Caddyfile` from templates
* Starting the full container stack (Vaultwarden, Caddy, AdGuard, Paperless-ngx, Grafana, Prometheus, cAdvisor, Beszel hub + agent)
* Setting up nightly backup scripts + cron jobs for Vaultwarden and Paperless

## Usage

```bash
ansible-playbook playbook.yml --ask-become-pass
ansible-vault encrypt ~/ansible-homeserver/group_vars/all/vault.yml --vault-password-file ~/.ansible-vault-pass
ansible-vault view ~/ansible-homeserver/group_vars/all/vault.yml --vault-password-file ~/.ansible-vault-pass
ansible-vault edit ~/ansible-homeserver/group_vars/all/vault.yml --vault-password-file ~/.ansible-vault-pass

docker run --rm -it \
  -v ~/docker/opencloud-config:/etc/opencloud \
  -v ~/docker/opencloud-data:/var/lib/opencloud \
  opencloudeu/opencloud:2 \
  idm resetpassword

docker run --rm -it \
  -v ~/docker/opencloud-config:/etc/opencloud \
  -v ~/docker/opencloud-data:/var/lib/opencloud \
  -e OC_CONFIG_DIR=/etc/opencloud \
  -e OC_DATA_DIR=/var/lib/opencloud \
  -e IDM_ADMIN_PASSWORD='YOUR_ADMIN_PASSWORD' \
  opencloudeu/opencloud:2 \
  init
```

Adjust variables (hostnames, ports, retention policy) in `group_vars/place-holder.yml` before running.

## Accessing services

Tailscale hostname, split by subpath:

https://{{ tailscale_hostname }}/vaultwarden
https://{{ tailscale_hostname }}/paperless
https://{{ tailscale_hostname }}/opencloud
https://{{ tailscale_hostname }}/beszel
https://{{ tailscale_hostname }}/grafana

Uptime Kuma and AdGuard Home are exposed separately on:
http://<tailscale_hostname>:{{ port }}

## File structure

```
.
File Structure
.
├── inventory.ini
├── playbook.yml
├── group_vars/
│   ├── all.yml
│   └── vault.yml
└── templates/
    ├── docker-compose.yml.j2
    ├── Caddyfile.j2
    ├── backup-vaultwarden.sh.j2
    ├── backup-paperless.sh.j2
    └── backup-opencloud.sh.j2
```
