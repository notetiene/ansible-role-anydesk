AnyDesk Ansible Role
====================

[![CI](https://img.shields.io/github/actions/workflow/status/notetiene/ansible-role-anydesk/ci.yml?branch=main&style=for-the-badge&label=CI)](https://github.com/notetiene/ansible-role-anydesk/actions/workflows/ci.yml)
![GitHub License](https://img.shields.io/github/license/notetiene/ansible-role-anydesk?style=for-the-badge)
[![Ansible Role](https://img.shields.io/ansible/role/d/notetiene/anydesk?style=for-the-badge)](https://galaxy.ansible.com/ui/standalone/roles/notetiene/anydesk/)

Installs [AnyDesk](https://anydesk.com) from its official APT repository,
manages the `anydesk` service and writes options to the system and per-user
`system.conf` files.

Supported platforms: Ubuntu jammy (22.04), noble (24.04) and resolute
(26.04); CI tests jammy and resolute. The repository setup runs on the Debian family only; on other
systems the role assumes a package manager that already knows `anydesk`.

Requirements
------------

- ansible-core 2.15 or later (the role uses
  `ansible.builtin.deb822_repository`).
- `become` access on the target. The role installs `gpg` and
  `python3-debian` itself.

Installation
------------

```sh
ansible-galaxy role install notetiene.anydesk
```

Or in `requirements.yml`:

```yaml
roles:
  - src: notetiene.anydesk
    version: 1.1.3
```

What the role does
------------------

1. Removes the files of earlier versions of the role
   (`anydesk_apt_legacy_files`). The old key in `/etc/apt/trusted.gpg.d/`
   was trusted for every repository.
2. Downloads the AnyDesk key, checks with `gpg` that it holds
   `anydesk_apt_key_id` and copies it to `anydesk_apt_keyring_file`. The
   play fails if the fingerprint does not match.
3. Writes `/etc/apt/sources.list.d/<anydesk_apt_filename>.sources`, signed by
   that key only. It does not use `apt-key`, which APT 3 removed.
4. Installs the `anydesk` package with `state: latest`, so every run
   upgrades AnyDesk to the newest version.
5. Sets the service state and start at boot. When the package or a system
   option changes, the handler sets `anydesk_restart_handler_state`.
6. Writes `anydesk_system_options` to `anydesk_system_conf_file` and each
   entry of `anydesk_system_options_per_user` to that user's file.

Role variables
--------------

### APT repository

| Variable | Default | Description |
|---|---|---|
| `anydesk_apt_key_url` | `https://keys.anydesk.com/repos/DEB-GPG-KEY` | URL of the repository key. |
| `anydesk_apt_key_id` | `06B5EA2FAE208E7CDA9761DCA2FB21D5A8772835` | Full fingerprint the downloaded key must hold. |
| `anydesk_apt_key_download_path` | `/tmp/anydesk-apt-key.asc` | Where the key is downloaded before the check. |
| `anydesk_apt_keyring_file` | `/etc/apt/keyrings/anydesk.asc` | Installed key, referenced by `Signed-By`. |
| `anydesk_apt_legacy_files` | the one-line source and keyrings of 1.0.x | Files removed before the source is written. |
| `anydesk_apt_repository` | `https://deb.anydesk.com` | Repository URI. |
| `anydesk_apt_codename` | `all` | Suite. |
| `anydesk_apt_channel` | `main` | Component. |
| `anydesk_apt_filename` | `anydesk-stable` | Name of the `.sources` file. |

### Service

| Variable | Default | Description |
|---|---|---|
| `anydesk_service_manage` | `true` | Manage the `anydesk` service. With `false`, the service task and the handler are skipped. |
| `anydesk_service_state` | `started` | State during the play; any [`service` state](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html#parameter-state). |
| `anydesk_service_enabled` | `true` | Start the service at boot. |
| `anydesk_restart_handler_state` | `restarted` | State the handler sets after an install, upgrade or system option change. Set it to `stopped` to keep a stopped service stopped. |

### Configuration

| Variable | Default | Description |
|---|---|---|
| `anydesk_system_conf_file` | `/etc/anydesk/system.conf` | System configuration file. |
| `anydesk_system_options` | `[]` | Options written to `anydesk_system_conf_file`. |
| `anydesk_system_options_per_user` | `[]` | Options written to each user's file. |

Each option is a dictionary with `name` and `value`, written as the line
`name=value`; an existing line for the same name is replaced. This README
does not list the options, since they change between AnyDesk versions.

Each entry of `anydesk_system_options_per_user` has:

- `user`: the user name (required);
- `options`: a list of options as above (required);
- `config_file`: the file to write (optional, default
  `~/.anydesk/system.conf` in the user's home). The file and its directory
  are owned by the user.

Example playbook
----------------

Install AnyDesk without starting it, and set `ad.security.interactive_access`
to `2` (which currently means: never allow remote control) for the system and
for one user:

```yaml
---
- hosts: desktops
  roles:
    - role: notetiene.anydesk
      anydesk_service_enabled: false
      anydesk_service_state: stopped
      anydesk_restart_handler_state: stopped
      anydesk_system_options:
        - name: ad.security.interactive_access
          value: 2
      anydesk_system_options_per_user:
        - user: jdoe
          options:
            - name: ad.security.interactive_access
              value: 2
```

Upgrading from 1.0.x
--------------------

1.1.0 replaces the `apt_key` setup. The first run removes
`/etc/apt/sources.list.d/anydesk-stable.list` and the old keyrings, then
writes `anydesk-stable.sources`. The play needs no change. The 1.0.x
defaults defined `anydesk_system_option` (singular) by mistake; a play that
did not set `anydesk_system_options` failed. 1.1.0 defines the right name.

Testing
-------

The Molecule scenario starts from the state 1.0.x left on a host, runs the
role, checks that a second run changes nothing and verifies the source, the
removed legacy files and the options. It needs Docker:

```sh
pip install ansible-core molecule "molecule-plugins[docker]" docker
MOLECULE_DISTRO=ubuntu2204 molecule test
MOLECULE_DISTRO=ubuntu2604 molecule test
```

GitHub Actions runs yamllint, ansible-lint and both scenarios on every push,
pull request and once a week. A tag also triggers the Galaxy import, which
needs the `GALAXY_API_KEY` repository secret.

License
-------

MIT
