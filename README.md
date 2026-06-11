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
    src: git@git.sr.ht:the-commits/kavita
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
| `kavita_docker_network` | `kavita` | Docker network name |
| `kavita_memory_max` | `512m` | Max memory for container |
| `kavita_memory_reservation` | `256m` | Memory reservation |
| `kavita_cpus_max` | `1` | Max CPU count |
| `kavita_cpus_reservation` | `0.5` | CPU reservation |

## Dependencies

Requires Docker Engine and Docker Compose. See
[`tjenamors-se-docker`](https://git.sr.ht/~the-commits/tjenamors-se-docker) and
[`tjenamors-se-docker-compose`](https://git.sr.ht/~the-commits/tjenamors-se-docker-compose).

## Example Playbook

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

## Resource Limits

By default, containers are limited to 512 MB RAM and 1 vCPU.
Adjust via role variables:

```yaml
kavita_memory_max: 1024m
kavita_cpus_max: "2"
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
