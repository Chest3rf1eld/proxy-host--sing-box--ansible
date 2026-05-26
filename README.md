[Read this in Russian](README.ru.md)

# sing-box Client Ansible

This repository automates the setup of a remote `sing-box` client.

In practical terms, it prepares a server that routes your traffic through an external peer so you can reach services that are unavailable from a specific country or region.

This repository is useful for:

- engineers who want a repeatable way to configure a remote `sing-box` client
- teams that want infrastructure setup described as code instead of manual actions
- non-technical stakeholders who need a simple explanation of the repository purpose

## What This Repository Does

After running the playbook, the target server is prepared to work as a `sing-box` client:

- the `sing-box` package is installed
- the client configuration is generated from variables
- the configuration is validated before deployment
- the `sing-box` service is enabled and started

## Typical Use Case

A typical scenario is using this host as an outbound client to connect to external services that are blocked or unavailable from your current country.

## Structure

- `site.yml` — main playbook
- `inventory/hosts.yml.example` — example inventory with the target host format
- `group_vars/sing_box.yml.example` — example `sing-box` client configuration
- `roles/sing_box/` — Ansible role that performs installation and configuration

## Local Files

These files should stay only on your machine and are ignored by Git:

- `inventory/hosts.yml`
- `group_vars/sing_box.yml`

Create them from the example files and replace the placeholder values with your real settings.

## Usage

1. Create local files from examples:
   - `cp inventory/hosts.yml.example inventory/hosts.yml`
   - `cp group_vars/sing_box.yml.example group_vars/sing_box.yml`
2. Edit local files with your real host and `sing-box` settings.
3. Run the playbook:

```bash
ansible-playbook site.yml
```

## What The Role Does

The `sing_box` role:

- optionally configures the official APT repository on Debian-based systems
- installs the `sing-box` package
- renders `/etc/sing-box/config.json`
- validates the configuration before applying it
- enables and starts the `sing-box` service

## Getting The Config

The easiest way to get a working `sing-box` configuration is to export it from Nekoray:

1. Open the specific peer in Nekoray.
2. Click `Share`.
3. Choose `Export sing-box config`.
4. Copy the relevant values into `group_vars/sing_box.yml`.

## Notes

- The repository uses `group_vars/sing_box.yml` for runtime configuration instead of keeping live values in role defaults.
- Keep secrets, host-specific values, and private SSH key paths only in local files.
