Anydesk Ansible Role
========================

![GitHub License](https://img.shields.io/github/license/notetiene/ansible-role-anydesk?style=for-the-badge)
![Ansible Role](https://img.shields.io/ansible/role/d/notetiene/anydesk?style=for-the-badge&link=https%3A%2F%2Fgalaxy.ansible.com%2Fnotetiene%2Fanydesk)
![Ansible Role](https://img.shields.io/ansible/role/d/notetiene/anydesk?style=for-the-badge&link=https%3A%2F%2Fgalaxy.ansible.com%2Fnotetiene%2Fanydesk)

This role installs AnyDesk on a Linux System. Currently only support Ubuntu 20.04, but patches are welcomed.

Requirements
------------

No requirements.

Role Variables
--------------

### AnyDesk service

- `anydesk_service_manage` \
  **Required**: no \
  **Default**: `true` \
  **Type**: boolean \
  Allows managing the AnyDesk service.
- `anydesk_service_state` \
  **Required**: no \
  **Default**: `started` \
  **Type**: Valid service state according to [ansible.builtin.service module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html#parameter-state).
  The state of the service on runtime (when executing playbook).
- `anydesk_service_enabled` \
  **Required**: no \
  **Default**: `true` \
  **Type**: boolean \
  Whether the service should start on boot.
- `anydesk_restart_handler_state` \
  **Required**: no \
  **Default**: `restarted` \
  **Type**: Valid service state according to [ansible.builtin.service module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html#parameter-state). \
  The state of the service when installing or updating AnyDesk.

### AnyDesk Configuration
- `anydesk_system_conf_file` \
  **Required**: no \
  **Default**: `/etc/anydesk/system.conf` \
  **Type**: string (file path) \
  File for AnyDesk system configuration.
- `anydesk_system_options` \
  **Required**: no \
  **Default**: empty \
  **Type**: list of dictionaries in the form:
    * `name`: the name of the option
    * `value`: the value of the option.
  List of options that the system.conf file will have.  Options will be put in the `anydesk_system_conf_file` file.  No documentation for options are provided here since they may change per AnyDesk version.
- `anydesk_system_options_per_user` \
  **Required**: no \
  **Default**: empty \
  **Type**: list of dictionaries containing the user name and list of options like `anydesk_system_options`:
    * `user`: the name of the user.
    * `options`: list of dictionaries in the form of `anydesk_system_options`.
    * `config_file`: custom configuration file path (`system.conf`) for the user.  Otherwise, `~/.anydesk/system.conf` is used. \
  List of options specefic to users.

Example Playbook
----------------

This playbook installs AnyDesk, but disable the service at boot and prevent it from starting (just installation).  It also sets the `ad.security.interactive_access` option to `2` (which currently means preventing remote control).

```yaml
---
- hosts: servers
  roles:
     - role: notetiene.anydesk
       anydesk_service_enabled: false
       anydesk_service_state: stopped
       anydesk_restart_handler_state: stopped
       anydesk_system_options:
         - name: ad.security.interactive_access
           value: 2
       anydesk_system_options_per_user:
         - user: etienne
           options:
             - name: ad.security.interactive_access
               value: 2
```

License
-------

MIT

