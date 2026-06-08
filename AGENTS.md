# Repository Guidelines

## Project Structure & Module Organization

This repository is an Ansible project for provisioning a remote `sing-box` client.

- `site.yml` is the main playbook and targets the `sing_box` inventory group.
- `ansible.cfg` points Ansible at `inventory/hosts.yml` and `roles/`.
- `inventory/hosts.yml.example` shows the expected host inventory format.
- `group_vars/sing_box.yml.example` contains runtime `sing-box` configuration examples.
- `roles/sing_box/` contains the role implementation:
  - `defaults/main.yml` for non-secret defaults.
  - `tasks/main.yml` for installation, config deployment, and service startup.
  - `templates/config.json.j2` for the generated `/etc/sing-box/config.json`.
  - `handlers/main.yml` for service restart behavior.

Keep local files such as `inventory/hosts.yml` and `group_vars/sing_box.yml` out of commits.

## Build, Test, and Development Commands

- `cp inventory/hosts.yml.example inventory/hosts.yml` creates a local inventory.
- `cp group_vars/sing_box.yml.example group_vars/sing_box.yml` creates local runtime vars.
- `ansible-playbook --syntax-check site.yml` verifies playbook syntax.
- `ansible-playbook site.yml --check` performs a dry run where supported by modules.
- `ansible-playbook site.yml` applies the role to the configured host.

The deployed config is validated by the template task with `sing-box check -c %s` before replacing the target file.

## Coding Style & Naming Conventions

Use two-space indentation for YAML. Quote file modes such as `"0644"` and `"0755"`. Prefer descriptive task names in sentence case, matching the existing role style. Prefix role variables with `sing_box_` to keep ownership clear, for example `sing_box_config_path` or `sing_box_install_package`.

For Jinja templates, keep rendering logic minimal and place configurable values in `defaults/main.yml` or `group_vars/sing_box.yml.example`.

## Testing Guidelines

There is no standalone automated test suite. Before submitting changes, run `ansible-playbook --syntax-check site.yml`. For task changes, also run `ansible-playbook site.yml --check` against a disposable or non-production host. If you change `config.json.j2`, verify with a real `sing-box` binary because deployment depends on `sing-box check`.

## Commit & Pull Request Guidelines

The current history uses short, imperative commits such as `chore: snapshot current state` and descriptive initial setup commits. Follow that pattern: keep the subject concise and explain the reason in the body when behavior changes.

Pull requests should include a summary of changes, any validation commands run, and notes about affected platforms or required variables. Do not include secrets, private hostnames, private SSH key paths, or generated local inventory files.

## Security & Configuration Tips

Treat `group_vars/sing_box.yml` as sensitive. Store only safe defaults in role defaults and examples. When adding new variables, update the example file and README guidance without exposing live peer credentials.
