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
SPDX-FileCopyrightText: 2024 MASH project contributors
SPDX-FileCopyrightText: 2024 Sergio Durigan Junior
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Pi-hole

This is an [Ansible](https://www.ansible.com/) role which installs [Pi-hole](https://pi-hole.net/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Pi-hole is a DNS sinkhole which blocks ads and trackers for every device that uses it as its DNS server, and comes with a web interface to manage it.

See the project's [documentation](https://docs.pi-hole.net/) to learn what Pi-hole does and why it might be useful to you.

> [!WARNING]
> Running a public DNS server is not advisable. You should install Pi-hole in a trusted local network, or set its port exposure (via the variables you can find on the section below about [opening ports](#open-ports)) so that you don't expose your DNS server publicly to the whole world.

## Prerequisites

### Open ports

Pi-hole uses the following ports on your server:

- `53` over **TCP**, controlled by `pihole_container_dns_tcp_bind_port` — used for DNS over TCP
- `53` over **UDP**, controlled by `pihole_container_dns_udp_bind_port` (which follows the TCP one by default) — used for DNS over UDP

By default, these ports are not exposed, as Pi-hole answers every client which reaches it. To expose them on the address of the server in your local network or VPN, add the following configuration to your `vars.yml` file, replacing `192.168.1.15` with that address:

```yaml
pihole_container_dns_tcp_bind_port: '192.168.1.15:53'
```

The address is required. Docker opens the ports in the server's firewall by itself, bypassing rules set with tools such as `ufw`, so `0.0.0.0:53` exposes the DNS server on **all network interfaces**, including public ones. It also fails on servers where `systemd-resolved` listens on port `53` (e.g. Ubuntu), unless its stub resolver is [disabled](https://docs.pi-hole.net/docker/tips-and-tricks/#disable-systemd-resolved-port-53).

## Adjusting the playbook configuration

To enable Pi-hole with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# pihole                                                               #
#                                                                      #
########################################################################

pihole_enabled: true

########################################################################
#                                                                      #
# /pihole                                                              #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Pi-hole you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
pihole_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

>[!NOTE]
> The `pihole_path_prefix` variable can be adjusted to host under a subpath (e.g. `pihole_path_prefix: /pihole`).

### Set the password

You also need to set the password for the web interface. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `YOUR_PASSWORD_HERE` with your own value.

```yaml
pihole_environment_variables_ftlconf_webserver_api_password: YOUR_PASSWORD_HERE
```

As it is set by the role, the password cannot be changed on the web interface.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `pihole_environment_variables_additional_variables` variable

Pi-hole reads any of its settings from a `FTLCONF_<section>_<setting>` environment variable, named after the setting in its `pihole.toml` (e.g. `FTLCONF_dns_upstreams` for `upstreams` under `[dns]`). A setting configured this way takes precedence over `pihole.toml`, and cannot be changed on the web interface. Refer to [this page](https://docs.pi-hole.net/docker/configuration/) for details.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, the web interface of Pi-hole becomes available at the specified hostname like `https://example.com`. Log in with the password you have set.

To have Pi-hole block ads and trackers, set the server as the DNS server of your devices, or of your router, so that it hands the server out to them.

Pi-hole forwards the queries it does not block to Google's DNS servers, unless you choose other upstream DNS servers under **Settings** → **DNS** on the web interface.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu pihole` (or how you/your playbook named the service, e.g. `mash-pihole`).
