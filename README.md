# Fluentd Ansible role

This Ansible role installs the [fluent-package](https://www.fluentd.org/) agent (fluentd v6, formerly `td-agent`) on Debian-based hosts, using the official codename-aware install script published by the Fluentd project.

- [Getting Started](#getting-started)
	- [Prerequisites](#prerequisites)
	- [Installing](#installing)
- [Role Variables](#role-variables)
- [Example Playbook](#example-playbook)

## Getting Started

These instructions will get you a copy of the role for your Ansible playbook. Once launched, it will install and configure a [fluentd](https://fluentd.io/) agent as a systemd service.

### Prerequisites

- Ansible 2.2.1.0 or later.
- A Debian-based target host (tested against Ubuntu `jammy` and `noble`).
- `curl` available or installable via `apt` (the role installs it if missing).

For testing purposes, [Molecule](https://molecule.readthedocs.io/) with [Vagrant](https://www.vagrantup.com/) as driver (with [landrush](https://github.com/vagrant-landrush/landrush) plugin) and [VirtualBox](https://www.virtualbox.org/) as provider.

### Installing

Create or add to your roles dependency file (e.g `requirements.yml`):

```yaml
- src: https://github.com/form3tech-oss/fluentd-role
  name: fluentd
  version: <tag, commit hash or branch name>
```

Install the role with the `ansible-galaxy` command:

```bash
ansible-galaxy install -p roles -r requirements.yml -f
```

## Role Variables

### Installation

| Variable | Default | Description |
|---|---|---|
| `fluentd_user` | `_fluentd` | System user created by the `fluent-package` install script and used to own config/log paths. |
| `fluentd_group` | `_fluentd` | System group created by the `fluent-package` install script. |
| `fluentd_secondary_groups` | `[]` | Extra groups `fluentd_user` is appended to (e.g. to allow reading log files owned by other users). |
| `fluentd_package_version` | `"6"` | Major `fluent-package` version line to install. |
| `fluentd_package_channel` | `"lts"` | Release channel of the install script (e.g. `lts`). |
| `fluentd_install_script_url` | codename-aware CNCF CDN URL | URL of the official install script, built from `ansible_distribution_release` (e.g. `jammy`, `noble`), `fluentd_package_version` and `fluentd_package_channel`. Override to pin an exact script if needed. |
| `fluentd_install_script_dest` | `/var/tmp/install-fluent-package.sh` | Local path the install script is downloaded to before execution. |
| `fluentd_install_marker_path` | `/opt/fluent/bin/fluentd` | Path used to detect whether `fluent-package` is already installed, so the install script only runs once. |
| `fluentd_plugins` | `[]` | List of `{name, version}` gem plugins to install via `fluent-gem` (e.g. `fluent-plugin-systemd`). |
| `fluentd_plugins_required_libs` | `[]` | List of additional `apt` packages required to build/install the plugin gems above. |

### Configuration

| Variable | Default | Description |
|---|---|---|
| `fluentd_conf_path` | `/etc/fluent` | Directory where `fluentd.conf` is rendered. |
| `fluentd_log_path` | `/var/log/fluent` | Directory used for fluentd's own logs. |
| `fluentd_log_file` | `{{ fluentd_log_path }}/fluent.log` | Full path to fluentd's own log file. |
| `fluentd_playbook_templates_path` | `{{ playbook_dir }}/templates/fluentd` | Path the calling playbook is expected to provide templates in, notably `fluent.conf.j2`. |

### Service

| Variable | Default | Description |
|---|---|---|
| `fluentd_service_state` | `started` | Desired state of the systemd service: `started`, `stopped`, `restarted`, `reloaded`. |
| `fluentd_service_enabled` | `yes` | Whether the service is enabled on boot. |
| `fluentd_service_environment` | `{}` | Dict of `ENV_NAME: value` entries written into `/etc/default/fluentd` via `lineinfile` (preserving the package stub). The `fluent-package` systemd unit loads this file via `EnvironmentFile=-/etc/default/fluentd`. |

> The calling playbook must provide its own `fluent.conf.j2` under `fluentd_playbook_templates_path`; this role only manages installation, ownership of the config/log directories, `/etc/default/fluentd` environment entries, and the systemd service lifecycle. It does not template the `fluentd.service` unit itself — that comes from the `fluent-package` apt package.

## Example Playbook

```yaml
- hosts: someserver
  roles:
    - role: fluentd
      fluentd_secondary_groups:
        - adm
      fluentd_plugins:
        - name: fluent-plugin-systemd
          version: "1.1.1"
        - name: fluent-plugin-record-modifier
          version: "2.2.1"
```

[Original readme](README.md.idealista)
