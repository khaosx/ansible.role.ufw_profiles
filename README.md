# ansible.role.ufw_profiles

Create and manage reusable UFW application profiles from data, then optionally allow selected profiles.

## Purpose

This role:
- Renders UFW app profiles into `/etc/ufw/applications.d`
- Refreshes UFW app cache when profile definitions change
- Optionally enables selected profiles with `ufw allow <profile>`

## Requirements

- Ubuntu host with UFW available
- `become: true` at play/role inclusion level
- `community.general` collection for the UFW module task

## Role Execution Model

This role is intended to run subordinately from a controlling playbook.

The controlling playbook should own:
- Baseline host provisioning
- Shared inventory and vault structure
- Global service/firewall policy inputs
- Role ordering and privilege model

## Control Playbook Contract

Role defaults live in `defaults/main.yml`.
Controller inventory should provide profile catalog data and profile-allow lists.

### Expected upstream variables

| Variable | Required | Description |
|---|---|---|
| `ufw_profiles_catalog` | yes | List of UFW application profile definitions to render |
| `ufw_profiles_allow_applications` | no | List of rendered profile names to allow |
| `ufw_profiles_update_cache` | no | Whether to run `ufw app update all` when profile files changed |

### Variable naming convention

Role-owned variables use the `ufw_profiles_` prefix.

## Data Model

Each `ufw_profiles_catalog` item should provide:
- `name` (required): profile section name
- `ports` (required): list of UFW ports/protocol entries (for example `"53"`, `"67/udp"`, `"443/tcp"`)
- `title` (optional)
- `description` (optional)
- `profile_filename` (optional; defaults to normalized `name`)

## Example Controller Play

```yaml
---
- name: Apply shared UFW profiles
  hosts: all
  become: true
  roles:
    - role: ufw_profiles
```

## License

MIT
