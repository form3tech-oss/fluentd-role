# Fluentd Ansible role

This Ansible role installs the [fluent-package](https://www.fluentd.org/) agent (fluentd v6, formerly `td-agent`) on Debian-based hosts, using the official codename-aware install script published by the Fluentd project.

- [Getting Started](#getting-started)
	- [Prerequisites](#prerequisites)
	- [Installing](#installing)
- [What this role does](#what-this-role-does)
- [Role Variables](#role-variables)
- [Example Playbook](#example-playbook)
- [Tags](#tags)

## Getting Started

These instructions will get you a copy of the role for your Ansible playbook. Once launched, it will install and configure a [fluentd](https://fluentd.io/) agent as a systemd service.

### Prerequisites

- Ansible 2.2.1.0 or later.
- A Debian-based target host (tested against Ubuntu `jammy` and `noble`).
- `curl` available or installable via `apt` (the role installs it if missing).

### Installing

Create or add to your roles dependency file (e.g. `requirements.yml`):

```yaml
- src: https://github.com/form3tech-oss/fluentd-role
  name: fluentd
  version: <tag, commit hash or branch name>
```

Install the role with the `ansible-galaxy` command:

```bash
ansible-galaxy install -p roles -r requirements.yml -f
```

## What this role does

1. **Install** — downloads and runs the official `fluent-package` install script for the host's Ubuntu/Debian codename, then optionally appends secondary groups to `_fluentd` and installs plugin gems via `fluent-gem`.
2. **Configure** — ensures `/etc/fluent` and `/var/log/fluent` exist and are owned by `_fluentd`, renders the playbook-supplied `fluent.conf.j2` to `/etc/fluent/fluentd.conf`, and upserts entries from `fluentd_service_environment` into `/etc/default/fluentd`.
3. **Service** — enables/starts (or stops) the `fluentd` systemd unit that ships with `fluent-package`.

This role does **not** template `fluentd.service`. The package unit already loads environment variables via:

```text
EnvironmentFile=-/etc/default/fluentd
```

so custom env vars (tokens, proxies, etc.) must go in `/etc/default/fluentd`, not in a custom unit file.

## Role Variables

### Installation

| Variable | Default | Description |
|---|---|---|
| `fluentd_user` | `_fluentd` | System user created by the `fluent-package` install script; used as owner of config/log paths. |
| `fluentd_group` | `_fluentd` | System group created by the `fluent-package` install script. |
| `fluentd_secondary_groups` | `[]` | Extra groups appended to `fluentd_user` (e.g. `systemd-journal` so fluentd can read the journal). |
| `fluentd_package_version` | `"6"` | Major `fluent-package` version line to install. |
| `fluentd_package_channel` | `"lts"` | Release channel of the install script (e.g. `lts`). |
| `fluentd_install_script_url` | codename-aware CNCF CDN URL | URL of the official install script, built from `ansible_distribution_release` (e.g. `jammy`, `noble`), `fluentd_package_version`, and `fluentd_package_channel`. Override to pin an exact script if needed. |
| `fluentd_install_script_dest` | `/var/tmp/install-fluent-package.sh` | Local path the install script is downloaded to before execution. |
| `fluentd_install_marker_path` | `/opt/fluent/bin/fluentd` | Path used as an Ansible `creates` marker so the install script only runs once. |
| `fluentd_plugins` | `[]` | List of `{name, version}` gem plugins to install via `/usr/sbin/fluent-gem` (e.g. `fluent-plugin-systemd`). |

### Configuration

| Variable | Default | Description |
|---|---|---|
| `fluentd_conf_path` | `/etc/fluent` | Directory where `fluentd.conf` is rendered. |
| `fluentd_log_path` | `/var/log/fluent` | Directory created/owned for fluentd's own logs (the package unit writes `fluentd.log` here by default). |
| `fluentd_playbook_templates_path` | `{{ playbook_dir }}/templates/fluentd` | Directory where the calling playbook must provide `fluent.conf.j2`. |

### Service

| Variable | Default | Description |
|---|---|---|
| `fluentd_service_state` | `started` | Desired state of the systemd service: `started`, `stopped`, `restarted`, `reloaded`. |
| `fluentd_service_enabled` | `yes` | Whether the service is enabled on boot. |
| `fluentd_service_environment` | `{}` | Dict of `ENV_NAME: value` entries upserted into `/etc/default/fluentd` with `lineinfile` (package stub keys such as `FLUENT_PACKAGE_OPTIONS` are preserved). File is kept `root:root` `0600`. Loaded by the package unit via `EnvironmentFile=-/etc/default/fluentd`. |

> The calling playbook **must** provide its own `fluent.conf.j2` under `fluentd_playbook_templates_path`.

## Example Playbook

```yaml
- hosts: someserver
  roles:
    - role: fluentd
      fluentd_secondary_groups:
        - systemd-journal
      fluentd_plugins:
        - name: fluent-plugin-systemd
          version: "1.1.1"
        - name: fluent-plugin-logzio
          version: "0.2.2"
      fluentd_service_environment:
        LOGZ_TOKEN: "PLACEHOLDER_LOGZ_TOKEN"
        HTTP_PROXY: "PLACEHOLDER_HTTP_PROXY"
        HTTPS_PROXY: "PLACEHOLDER_HTTPS_PROXY"
        NO_PROXY: "PLACEHOLDER_NO_PROXY"
      fluentd_service_state: stopped
      fluentd_service_enabled: false
```

## Tags

| Tag | Tasks |
|---|---|
| `install` | Download/run the install script, secondary groups, plugin gems |
| `configure` | Config/log directories, `fluentd.conf`, `/etc/default/fluentd` |
| `service` | systemd enable/state |

[Original Idealista readme](README.md.idealista)
