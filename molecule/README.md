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

There are two testing scenarios available.

Neither of them stops at "the systemd service is active" — `Restart=always` makes a crash-looping container look active, and an uninstalled WordPress answers `200` or `302` on every path there is (`/`, `/wp-login.php`, `/wp-json/`, and paths that do not exist), so neither the unit state nor an HTTP response says anything on its own.

What both scenarios do instead:

- establish, before anything is installed, that WordPress answers with exactly the redirect to `/wp-admin/install.php`. A WordPress that cannot reach its database answers `500` instead, so this is what proves the database the role configured is reachable with the credentials the role configured — and it is what makes everything asserted after the install step meaningful, because it establishes that none of it could have passed a moment earlier
- drive WordPress's own installer over HTTP, non-interactively
- assert that the running WordPress is the version `wordpress_version` pins (WordPress reports `7.1` where the image is tagged `7.1.0`, so the comparison allows the shorter form as a prefix)
- read the role's `uploads.ini` and `env` back out of the *serving* process, through a small PHP file fetched over HTTP — which is what distinguishes "the file is on disk" from "the process is running with it"
- find the installed site in MariaDB, under the table prefix the role passed through, and then render a post that was inserted straight into MariaDB. Both directions have to work, which rules out a WordPress serving from anywhere other than the database the role pointed it at
- assert at the end that the service has not restarted

Every value the scenarios assert their way back to (upload size, timezone, table prefix, database name, credentials, ports) is deliberately neither a default of the role nor a default of WordPress, and differs between the two scenarios, so a configuration file that never reached the process shows up as a failure rather than as a pass.

### `default`

Tests a standard WordPress installation against a MariaDB installed by [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and reached over the Unix socket the role bind-mounts.

### `database-tcp`

Tests the other branch of `wordpress_database_socket_enabled`: MariaDB reached over TCP, with the socket mount absent — asserted explicitly, since a WordPress that quietly kept using a socket would pass a verification that only looked at whether the site works.

This scenario also runs WordPress on a container port that is not `80`. The image hardcodes `Listen 80` and `<VirtualHost *:80>`, so `wordpress_container_http_port` only means anything because the role renders `../templates/apache-ports.conf.j2` and `../templates/apache-vhost.conf.j2` and mounts them over the image's own files. Without that, the variable would relabel the port for Traefik and for the host port publishing while the process kept listening on 80.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
molecule test --scenario-name database-tcp
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
