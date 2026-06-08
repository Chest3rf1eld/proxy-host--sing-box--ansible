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

The example configuration also enables a TUN inbound. It intercepts outbound traffic from the server itself, proxies only ChatGPT/OpenAI/Codex CLI domains through the `proxy` outbound, and sends all other traffic directly through the `direct` outbound.

For Codex CLI, this covers both common modes: ChatGPT login through `chatgpt.com` and API-key mode through `api.openai.com`, because `api.openai.com` is covered by the `openai.com` suffix rule.

The local mixed proxy on `127.0.0.1:2080` stays available, but it now follows the same routing rules instead of forcing all traffic through the proxy.

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

After applying the playbook, you can check Codex CLI on the target server:

```bash
codex --version
codex exec "ping"
```

To test the local mixed proxy directly without relying on TUN interception:

```bash
HTTPS_PROXY=http://127.0.0.1:2080 codex exec "ping"
```

## What The Role Does

The `sing_box` role:

- optionally configures the official APT repository on Debian-based systems
- installs `sing-box` from the APT package on Debian-based systems
- installs `sing-box` from the GitHub release archive on RedHat-family systems such as Sangoma OS
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
- TUN mode needs permission to create a network interface and update routes; the stock `sing-box` systemd service usually runs with the required privileges.
- If Codex CLI signs in through Google, Microsoft, or Apple, OpenAI traffic is still proxied, but the external OAuth provider may need a separate routing rule.
- For hosts without direct internet access where `sing-box` is already installed, set `sing_box_install_package: false`; the role will skip APT repository setup, `apt-get update`, and package installation.
- On Sangoma OS, the default `sing_box_install_method: auto` uses the release archive because the standard package repositories do not provide `sing-box`. Pin a different release with `sing_box_archive_version`.
