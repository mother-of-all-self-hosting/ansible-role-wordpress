<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Matomo

This is an [Ansible](https://www.ansible.com/) role which installs [Matomo](https://matomo.org/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Matomo is a leading open-source web analytics platform that gives you full data ownership.

See the project's [documentation](https://matomo.org/guides/) to learn what Matomo does and why it might be useful to you.

## Prerequisites

To run a Matomo instance it is necessary to prepare a [MySQL](https://www.mysql.com/) compatible database server.

If you are looking for an Ansible role for [MariaDB](https://mariadb.org/), you can check out [this role (ansible-role-mariadb)](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Matomo with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# matomo                                                               #
#                                                                      #
########################################################################

matomo_enabled: true

########################################################################
#                                                                      #
# /matomo                                                              #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Matomo you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
matomo_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Matomo under a subpath (by configuring the `matomo_path_prefix` variable) does not seem to be possible due to Matomo's technical limitations.

### Set variables for connecting to a MySQL-compatible database server

To have the Matomo instance connect to your MySQL-compatible database server, add the following configuration to your `vars.yml` file.

```yaml
matomo_database_username: YOUR_MYSQL_SERVER_USERNAME_HERE
matomo_database_password: YOUR_MYSQL_SERVER_PASSWORD_HERE
matomo_database_name: YOUR_MYSQL_SERVER_DATABASE_NAME_HERE
```

### Configuring connection to the database server (optional)

By default the role is configured to establish connection with the the database server via the Unix socket. You can mount the Unix socket by adding the following configuration to your `vars.yml` file:

```yaml
# Specify the path to the the database Unix socket path on the host (bind-mount source)
matomo_database_socket_path_host: ""
```

Setting it enables to connect to the the database server via Unix socket mounted in the container at `/run-mysqld/mysqld.sock`.

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to the database server via a Unix socket
matomo_database_socket_enabled: false

matomo_database_hostname: YOUR_MYSQL_SERVER_HOSTNAME_HERE
```

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `matomo_environment_variables_additional_variables` variable

See [the image's documentation](https://hub.docker.com/_/matomo/#matomo-installation) for a complete list of Matomo's config options that you could put in `matomo_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Matomo becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and follow the set up wizard.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu matomo` (or how you/your playbook named the service, e.g. `mash-matomo`).
