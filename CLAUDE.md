# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Linting

```bash
ansible-lint
```

CI runs ansible-lint via the `.gitlab-ci.yml` pipeline (GitLab component). The `.ansible-lint` config skips the `role-name` rule.

## Role Overview

This is an Ansible role (`common`) that configures baseline system settings on Debian/Ubuntu hosts. It depends on the `apt` role for package installation.

### What it configures

- **Hostname** — sets from `inventory_hostname_short`, updates `/etc/hostname` and `/etc/hosts`
- **Environment** — deploys a static `/etc/environment` with a hardened `PATH`
- **Locale** — generates locale and writes `/etc/default/locale`
- **Timezone** — sets via `community.general.timezone`
- **SSH hardening** — disables root login and password auth (with `sshd -t` validation before restart)
- **DNS resolver** — optionally rewrites `/etc/resolv.conf` (opt-in via `common_resolv_conf_edit`)
- **Packages** — three lists merged and passed to the `apt` role dependency: `common_packages` (base), `common_packages_env` (environment-specific), `common_packages_host` (host-specific)

All tasks are tagged `common` plus a sub-tag (e.g., `common:hostname`, `common:locale`, `common:ssh`).

### Key variables (see `defaults/main.yml`)

| Variable | Default |
|---|---|
| `common_locale` | `en_US.UTF-8` |
| `common_timezone` | `Etc/UTC` |
| `common_ssh_password_auth` | `no` |
| `common_ssh_root_access` | `no` |
| `common_resolv_conf_edit` | `false` |
| `common_resolv_conf_ns` | `[8.8.8.8, 8.8.4.4]` |

### Templates

- `templates/etc/hosts.j2` — localhost entries + `inventory_hostname`/`inventory_hostname_short` + loop over `common_hosts_entries`
- `templates/etc/default/locale.j2` — `LANG` / `LANGUAGE`
- `templates/etc/resolv.conf.j2` — nameservers + optional search domains
