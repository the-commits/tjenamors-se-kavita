[![sourcehut](https://img.shields.io/badge/sourcehut-~the--commits/tjenamors--se--kavita-2d6b9e?logo=sourcehut)](https://git.sr.ht/~the-commits/tjenamors-se-kavita)
[![GitHub mirror](https://img.shields.io/badge/GitHub-the--commits/tjenamors--se--kavita-181717?logo=github)](https://github.com/the-commits/tjenamors-se-kavita)

# Deploy Kavita eBook reader

An Ansible role to deploy Kavita via Docker Compose

## Requirements

- Ansible 2.14+
- Root/sudo access on target hosts
- Linux (x86_64 or aarch64)
- Docker Engine and Docker Compose installed on target hosts
- Minimum server resources:
  - 512 MB RAM
  - 1 vCPU
  - 10 GB disk space

## Installation

### From Ansible Galaxy (once published)

```bash
ansible-galaxy install tjenamors.kavita
```

### From source

Add to your `requirements.yml`:

```yaml
---
roles:
  - name: kavita
    src: git@git.sr.ht:~the-commits/tjenamors-se-kavita
    scm: git
    version: main
```

Then install:

```bash
ansible-galaxy install -r requirements.yml -p roles/
```

## Role Variables

All variables are prefixed with `kavita_`. See [`defaults/main.yml`](defaults/main.yml) for the full list.

| Variable | Default | Description |
|---|---|---|
| `kavita_base_dir` | `/opt/kavita` | Installation directory |
| `kavita_image` | `jvmilazz0/kavita:latest` | Docker image to use |
| `kavita_container_name` | `kavita` | Container name |
| `kavita_docker_network` | `kavita` | Internal Docker network name |
| `kavita_traefik_network` | `traefik` | External Traefik network name |
| `kavita_port` | `5000` | Container port |
| `kavita_tz` | `UTC` | Container timezone |
| `kavita_memory_max` | `512m` | Max memory for container |
| `kavita_memory_reservation` | `256m` | Memory reservation |
| `kavita_cpus_max` | `1` | Max CPU count |
| `kavita_cpus_reservation` | `0.5` | CPU reservation |
| `kavita_traefik_enabled` | `false` | Enable Traefik reverse proxy |
| `kavita_traefik_domain` | `kavita.example.com` | Traefik domain |
| `kavita_shutdown_timeout` | `30` | Systemd shutdown timeout |

## Dependencies

Requires Docker Engine and Docker Compose. See
[`tjenamors-se-docker`](https://git.sr.ht/~the-commits/tjenamors-se-docker) and
[`tjenamors-se-docker-compose`](https://git.sr.ht/~the-commits/tjenamors-se-docker-compose).

Optionally requires [`tjenamors-se-traefik`](https://git.sr.ht/~the-commits/tjenamors-se-traefik)
when `kavita_traefik_enabled` is `true`.

## Traefik Integration

When `kavita_traefik_enabled` is `true`, Kavita joins the shared Traefik Docker
network and publishes Traefik labels for automatic routing. The container port
is not exposed directly — all traffic goes through Traefik on port 443 with
Let's Encrypt TLS.

```yaml
kavita_traefik_enabled: true
kavita_traefik_domain: bibliotek.tjenamors.se
```

## Example Playbook

### Standalone (no Traefik)

```yaml
- hosts: all
  become: true
  vars:
    kavita_base_dir: /opt/kavita
  roles:
    - role: docker
    - role: docker_compose
    - role: kavita
```

### Behind Traefik

```yaml
- hosts: all
  become: true
  vars:
    kavita_traefik_enabled: true
    kavita_traefik_domain: bibliotek.tjenamors.se
    kavita_base_dir: /opt/kavita
  roles:
    - role: docker
    - role: docker_compose
    - role: traefik
    - role: kavita
```

## Resource Limits

By default, containers are limited to 512 MB RAM and 1 vCPU.
Adjust via role variables:

```yaml
kavita_memory_max: 2048m
kavita_cpus_max: "1"
```

## Service Management (systemd)

On systemd-based distros, manage the service like any other:

```bash
sudo systemctl status kavita
sudo systemctl start kavita
sudo systemctl stop kavita
sudo systemctl restart kavita
sudo journalctl -u kavita
```

## Local Development

### Prerequisites

```bash
pip install molecule molecule-plugins[vagrant]
vagrant plugin install vagrant-libvirt
```

### Running Tests

```bash
molecule test
```

## License

AGPL-3.0 — see [LICENSE](LICENSE) for details.
