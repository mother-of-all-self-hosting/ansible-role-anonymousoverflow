<!--
SPDX-FileCopyrightText: 2018-2026 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently these testing scenarios are available:

### `default`

Tests a standard AnonymousOverflow installation, installed from the published container image.

It first establishes what an AnonymousOverflow the role has *not* configured does, so that everything it checks afterwards can be credited to the role rather than to the image:

- the image on its own refuses to start at all — AnonymousOverflow panics without `APP_URL` and `JWT_SIGNING_SECRET`, so it cannot serve a wizard, a placeholder or anything else that a naive probe might mistake for a working install
- an instance given only those two, with values of its own, listens on AnonymousOverflow's own default port, renders the `auto` theme, prefixes its redirects with its own `APP_URL`, rejects a token signed with the role's signing secret, and still rate limits

Against that baseline it checks that every value the role configures reached the process — the port AnonymousOverflow announces on startup, the theme it renders into its home page, the base URL it prefixes a resolved shortened link with, the signing secret its image proxy verifies tokens against, and the rate limiter it was told to turn off — that the container runs the image `anonymousoverflow_version` names, that the container is shaped the way the systemd unit says (read-only, unprivileged, all capabilities dropped, with the role's additional volume mounted), and that it does not restart while all of that is being read.

Nothing in it depends on StackOverflow answering. Two of the values above (`APP_URL` and `JWT_SIGNING_SECRET`) only show up on a surface that follows an outbound request, so the scenario stands a stub in StackOverflow's place: an nginx holding a certificate for `www.stackoverflow.com` signed by an authority created during preparation, which AnonymousOverflow reaches because the stub joins the role's own container network under the `www.stackoverflow.com` network alias, and trusts because `SSL_CERT_FILE` points Go's TLS trust at that authority. The stub returns nothing resembling real StackOverflow data and is not asked to.

### `default-selfbuild`

Tests an AnonymousOverflow installation that builds the container image from AnonymousOverflow's own `Dockerfile` rather than pulling it.

It deliberately does not repeat the `default` scenario's work. What it checks is what only it can: that the running image was built here (it carries no repository digest) under the self-build name, that the checkout it was built from sits on the ref `anonymousoverflow_version` names, and that the running process reports the version those sources compile into `config/version.go`.

Because it compiles AnonymousOverflow from source, CI runs it only when the version being built, the tasks doing the building or the scenario itself changed — or on demand through the workflow's `workflow_dispatch` trigger.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
