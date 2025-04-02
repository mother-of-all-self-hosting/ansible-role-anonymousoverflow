<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up AnonymousOverflow

This is an [Ansible](https://www.ansible.com/) role which installs [AnonymousOverflow](https://anonymousoverflow.org) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

AnonymousOverflow is a set of PHP scripts that will allow you to run Your Own URL Shortener, on your server.

See the project's [documentation](https://anonymousoverflow.org/docs) to learn what AnonymousOverflow does and why it might be useful to you.

## Prerequisites

To run a AnonymousOverflow instance it is necessary to prepare a [MySQL](https://www.mysql.com/) compatible database server.

If you are looking for an Ansible role for [MariaDB](https://mariadb.org/), you can check out [this role (ansible-role-mariadb)](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

See [here](https://anonymousoverflow.org/docs#server-requirements) on the official documentation to check server requirements.

## Adjusting the playbook configuration

To enable AnonymousOverflow with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# anonymousoverflow                                                    #
#                                                                      #
########################################################################

anonymousoverflow_enabled: true

########################################################################
#                                                                      #
# anonymousoverflow                                                    #
#                                                                      #
########################################################################
```

### Set the hostname

To enable AnonymousOverflow you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
anonymousoverflow_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting AnonymousOverflow under a subpath (by configuring the `anonymousoverflow_path_prefix` variable) does not seem to be possible due to AnonymousOverflow's technical limitations.

### Set the admin username and password

You also need to create an instance's user to access to the admin UI after installation. To create one, add the following configuration to your `vars.yml` file. Make sure to replace `YOUR_ADMIN_USERNAME_HERE` and `YOUR_ADMIN_PASSWORD_HERE`.

```yaml
anonymousoverflow_environment_variable_user: YOUR_ADMIN_USERNAME_HERE
anonymousoverflow_environment_variable_pass: YOUR_ADMIN_PASSWORD_HERE
```

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `anonymousoverflow_environment_variables_additional_variables` variable

For a complete list of AnonymousOverflow's config options that you could put in `anonymousoverflow_environment_variables_additional_variables`, see its [environment variables](https://anonymousoverflow.org/docs/guide/essentials/configuration).

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, AnonymousOverflow's admin UI is available at the specified hostname with `/admin/` such as `example.com/admin/`.

First, open the page with a web browser to complete installation on the server by clicking "Install AnonymousOverflow" button. After that, click the anchor link "AnonymousOverflow Administration Page" to log in with the username (`anonymousoverflow_environment_variable_user`) and password (`anonymousoverflow_environment_variable_pass`).

The help file is available at `example.com/readme.html`.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu anonymousoverflow` (or how you/your playbook named the service, e.g. `mash-anonymousoverflow`).
