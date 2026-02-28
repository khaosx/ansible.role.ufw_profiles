# Role: ufw_profiles

## Description

Manage reusable UFW application profiles by rendering profile definitions into `/etc/ufw/applications.d`, optionally refreshing the UFW app cache, and optionally allowing selected profiles.

## Requirements

- Ansible >= 2.14
- Collection: `community.general`
- Target OS: Ubuntu, Debian (with UFW installed)

## Privilege Escalation

requires_become: true

## Role Variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `ufw_profiles_catalog` | `[]` | No | List of profile definitions to render. Each item supports `name` (str), `ports` (list[str]), and optional `title`, `description`, `profile_filename`. |
| `ufw_profiles_allow_applications` | `[]` | No | List of profile names to allow with `community.general.ufw` after rendering. |
| `ufw_profiles_update_cache` | `true` | No | Whether to run `ufw app update all` when profile templates changed. |

## Dependencies

None.

## Outbound Artifacts

- Ensures directory `/etc/ufw/applications.d` exists.
- Renders managed UFW profile files in `/etc/ufw/applications.d/`.
- Optionally runs `ufw app update all` when profile files changed.
- Optionally applies allow rules for profile names via `community.general.ufw`.

## Capabilities

- Render one or more reusable UFW application profiles from structured inventory data.
- Normalize output filenames automatically when `profile_filename` is omitted.
- Gate cache refresh behind change detection.
- Apply optional allow rules from a declared profile name list.

## Check Mode Behavior

- The cache-refresh command task is skipped when no profile changes are detected.
- No tasks are gated with `when: not ansible_check_mode`.

## Idempotency

True. A second run with unchanged inputs should report no changes except where external UFW state is changed outside Ansible.

## Atomic

False. The role applies a sequence of operations (directory, files, cache update, allow rules) and does not provide all-or-nothing transactional rollback.

## Rollback

No automatic rollback is implemented.

## Required Credentials / Privileges

- Root privileges (`become: true`) to manage `/etc/ufw/applications.d` and firewall rules.

## Supported Platforms

- Ubuntu
- Debian

## Example Playbook

```yaml
---
- name: Apply shared UFW profiles
  hosts: firewall_hosts
  gather_facts: false
  become: true
  roles:
    - role: khaosx.homelab.ufw_profiles
      vars:
        ufw_profiles_catalog:
          - name: "DNS"
            title: "Domain Name Service"
            description: "Allow DNS traffic"
            ports:
              - "53"
              - "53/udp"
          - name: "HTTPS"
            ports:
              - "443/tcp"
        ufw_profiles_allow_applications:
          - "DNS"
          - "HTTPS"
        ufw_profiles_update_cache: true
```

## License

MIT

## Author

khaosx
