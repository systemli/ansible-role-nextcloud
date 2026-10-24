# AGENTS.md

Ansible role that installs, upgrades and configures Nextcloud (and its apps) on Debian (bookworm, trixie).

## Layout

- `defaults/main.yml`: all role variables with defaults
- `tasks/`: `main.yml` plus `tasks/nextcloud/` (download, install, upgrade, apps, patches, notify-push, backup, ...)
- `library/nextcloud_app.py`: custom module for managing Nextcloud apps
- `templates/`, `files/`, `handlers/`, `vars/`: supporting role content
- `molecule/default/`: integration test (Docker, depends on `systemli.mariadb` from `requirements.yml`)
- `requirements.txt` (Python, e.g. `jmespath`) and `requirements.yml` (roles and collections)

## Checks

- `ansible-lint .` must pass (config in `.ansible-lint`, which excludes `molecule/` and `.github/`)
- `yamllint .` uses `.yamllint`
- `molecule test` runs the role in a container; CI runs it on Debian 12 and 13 via `systemli/github-ansible-workflow`
- `ansible-lint` creates `.ansible/`; it is git ignored, never commit it

## Conventions

- Use fully qualified module names and `true`/`false` booleans
- Give every task a `name` and the matching `role:nextcloud:*` tags
- New or changed role variables need an entry in `defaults/main.yml` and the README

## Commits and PRs

- Write in English, use conventional commit titles (`feat:`, `fix:`, `chore:`, `ci:`; `!` for breaking changes)
- The PR title sets the release-drafter label (`enhancement`, `bug`, `chore`, `breaking`); a breaking change bumps the major version
- Open PRs as draft and fill in `.github/pull_request_template.md`: what changed and why, plus a breaking changes section if needed
- Do not list changed files or claim test results in the description; CI covers that
