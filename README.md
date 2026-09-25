# Home Server Ansible Playbook

Rebuilds a Vaultwarden + Caddy + AdGuard Home + Paperless-ngx + Grafana + Prometheus + cAdvisor + Beszel stack on a fresh Ubuntu machine. Turns a set of manual SSH steps into a repeatable, version-controlled setup.

Before you run this against your own machine: replace the placeholder values in `inventory.ini` and `group_vars/all.yml` (IP address, username, Tailscale hostname, Paperless secrets, Beszel token/key) with your own. This repo ships with generic placeholders on purpose so it's safe to keep public.

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
```

Adjust variables (hostnames, ports, retention policy) in `group_vars/all.yml` before running.

## Accessing services

Vaultwarden, Paperless, Beszel, and Grafana are all served through Caddy under one Tailscale hostname, split by subpath:

* Vaultwarden: `https://{{ tailscale_hostname }}/vaultwarden`
* Paperless: `https://{{ tailscale_hostname }}/paperless`
* Beszel: `https://{{ tailscale_hostname }}/beszel`
* Grafana: `https://{{ tailscale_hostname }}/grafana`

AdGuard's web UI is exposed on its own port (`{{ adguard_web_port }}`) rather than a subpath.

## Monitoring

This stack includes two separate monitoring approaches:

**Beszel** — a lightweight, self-hosted monitoring platform. Consists of two pieces:

* **beszel** (hub) — serves the web dashboard, bound to `127.0.0.1:8090` on the host and reverse-proxied by Caddy under `/beszel`. Stores historical metrics in a local volume (`beszel-data`).
* **beszel-agent** — runs in `network_mode: host` to report accurate network stats, and mounts the Docker socket (read-only) to collect per-container CPU/memory/network stats. Communicates with the hub over a Unix socket (`beszel-socket`) rather than exposing a network port.

The hub requires an `APP_URL` environment variable set to its full subpath URL (`https://{{ tailscale_hostname }}/beszel`) so its frontend correctly resolves asset paths when served behind a reverse proxy.

**Grafana + Prometheus + cAdvisor** — the heavier, more customizable stack:

* **cAdvisor** — exposes container-level metrics. Configured with `--docker_only=true` and `--disable_metrics=...` to only collect CPU/memory and skip the heavier disk/network/process metrics.
* **Prometheus** — scrapes cAdvisor on an internal Docker network and stores the time series. Its own UI/API is not exposed through Caddy; access it directly on its container port if needed for debugging (`http://<host>:9090`).
* **Grafana** — the only monitoring UI exposed through Caddy, under `/grafana`. Prometheus is added as a data source (`http://prometheus:9090`) and dashboards are imported from grafana.com (e.g. dashboard ID `14282` for cAdvisor).

The `prometheus-data` directory must be owned by the same UID the Prometheus container runs as (see `user:` in `docker-compose.yml.j2`) or it will fail to start with a permissions error on its storage directory.

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
    ├── prometheus.yml.j2
    ├── backup-vaultwarden.sh.j2
    └── backup-paperless.sh.j2
```
