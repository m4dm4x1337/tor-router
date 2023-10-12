# Tor-Router

## Description

**Tor-Router** is a transparent TOR proxy and DNS proxy with routing capabilities. It uses your operating system's packet filtering facility to redirect outbound connections into a transparent proxy. There are two common use cases for Tor's transparent routing functionality.

The first use case is routing all traffic on a standalone machine through Tor. Once this is set up, every network application will make its TCP connections through Tor, ensuring that no application will be able to reveal your IP address by connecting directly.

The second use case is creating an anonymizing middlebox, such as a Raspberry Pi WiFi hotspot, that intercepts traffic from other machines and redirects it through Tor.

## Installation

Download the Debian package and run this commands:

```bash
sudo dpkg -i tor-router_1.0_all.deb || sudo apt-get -f install
```

The `tor-router` package depends on the `tor` package which will be installed if it was not previously present on your system. You can stop and disable the `tor.service` as the service is not needed by Tor-Router:

```bash
sudo systemctl stop tor.service
sudo systemctl disable tor.service
```

Since the Tor-Router Debian package only consists of configuration files and scripts, it is architecture-independent and can be installed on any Debian-based system on which the `tor` package is available.

## Usage

To start the `tor-router.service` run this command:

```bash
sudo systemctl start tor-router.service
```

When you start Tor-Router for the very first time, or after a longer period of inactivity, it may take a few seconds until the TOR Bootstrapping is complete. To monitor the status of the tor-router service run this command:

```bash
watch -c -n1 SYSTEMD_COLORS=1 systemctl status tor-router.service --full --lines=10 --no-pager
```

Wait until the line **Bootstrapped 100%: Done** appears. After that everything should work. All your traffic will now be sent over the TOR network. No application will be able to reveal your IP address by connecting directly.


## Change IP address

You can change your IP address at any time by running the following command:

```bash
sudo systemctl reload tor-router.service
```

To check whether your IP address has actually changed, you can run the following command:

```bash
/usr/share/tor-router/sbin/whatsmyip
```

If you have IPv6 support enabled you can also use this command:

```bash
/usr/share/tor-router/sbin/whatsmyip6
```


## Configuration

The Tor-Router configuration can be customized by editing the `/etc/default/tor-router` file. The configuration file contains various options that control the behavior of Tor-Router. Below are the configurable variables and their explanations:

- **DPORTS** (Allowed destination ports): Comma-separated list of port numbers, service names, or ranges of port numbers. By default only http (port 80) and https (port 443) are allowed for outgoing connections.

- **LPORTS** (Allowed listen ports): Comma-separated list of port numbers, service names, or port ranges. By default only ssh (port 22) is allowed for incoming connections.

- **WHITELIST** (Allow direct access to networks): This variable whitelists specific IP addresses or CIDR's. Use `@AUTO` to automatically whitelist local networks.

- **BLACKLIST** (Deny access to networks): Comma-separated list of IP addresses or CIDR's. Use `@AUTO` to block unreachable networks that are not part of the global Internet.

- **ENABLE_IPV6** (Enable IPv6 when tor-router is running): Set to "yes" or "no" to enable or disable IPv6 routing. You can use `@AUTO` to make this depend on `net.ipv6.conf.default.disable_ipv6`.

- **PREFER_IPV6** (Prefer IPv6 when tor-router is running): Set to "yes" or "no" to indicate your preference for IPv6 routing.

- **ROUTER** (Allow routing of traffic from network clients): Set to "yes" or "no" to enable or disable the routing of traffic from network clients. You can use `@AUTO` to make this depend on `net.ipv4.conf.default.forwarding` and `net.ipv6.conf.default.forwarding`.

- **CONNECTION_LOG** (Log new connections): Set to "yes" or "no" to control whether new connections are logged to `/var/log/tor-router/`.

- **REJECT_LOG** (Log rejected connections): Set to "yes" or "no" to log rejected connections to `/var/log/tor-router/`.

- **VERBOSE** (Log iptables rules, ipset rules, and sysctl changes): Set to "yes" or "no" to enable or disable verbose logging. This logs detailed information about iptables rules, ipset rules, and sysctl changes.

- **VIRTUAL_ADDR4** and **VIRTUAL_ADDR6** (Tor's VirtualAddrNetworkIPv4 and VirtualAddrNetworkIPv6): Use `@AUTO` to automatically choose the address. These variables specify the virtual address networks for Tor.

- **SOCKS_PORT** (Tor-Router SocksPort): Port number 8050 is used by default.

- **TRANS_PORT** (Tor-Router TransPort): Port number 8040 is used by default.

- **DNS_PORT** (Tor-Router DNSPort): Port number 8053 is used by default.

- **GEOIP_EXCLUDE_UNKNOWN** (Exclude nodes with unknown geo IP location): Set to "0" or "1" to exclude nodes with an unknown geo IP location.

- **STRICT_NODES** (Strict nodes requirement): If set to "1," one of the following *_NODES options becomes a requirement.

- **ENTRY_NODES** (Use specific countries as entry nodes): Comma-separated list of specific countries to use as entry nodes.

- **EXIT_NODES** (Use specific countries as exit nodes): Comma-separated list of specific countries to use as exit nodes.

- **EXCLUDE_NODES** (Exclude nodes in specific countries): Comma-separated list of specific countries to exclude nodes from.

- **EXCLUDE_EXIT_NODES** (Exclude exit nodes in specific countries): Comma-separated list of specific countries to exclude as exit nodes.


## View full logs

To view the full logs since the service was last started you can use the following command:

```bash
sudo journalctl -aelq -u tor-router -n 1000 --no-pager _SYSTEMD_INVOCATION_ID="$(systemctl show -p InvocationID --value tor-router)"
```

If you made a mistake in your configuration (such as specifying port number 70000 among the allowed ports), you will see it as red blinking text in the full logs. You can't miss it.

## Plugins

Plugins can be created as simple shell scripts that are sourced by Tor-Router. Check out the existing plugins in the following directories to see examples:

```bash
/etc/tor-router/tr-pre-up.d/
/etc/tor-router/tr-post-up.d/
/etc/tor-router/tr-pre-down.d/
/etc/tor-router/tr-post-down.d/
```

Plugins are only loaded if they belong to the root user and are executable.

## Graphical user interface (GUI)

There are **two** graphical user interfaces!

The first is an app indicator for the GNOME desktop that allows you to easily start/stop Tor-Router. You can also see whether Tor-Router is running or not based on the color of the icon and the label text next to the icon.

The second is a web GUI that can display a lot of additional information and enable remote management and monitoring of Tor-Router.

## Internationalization (i18n)

Tor-Router and its GUIs are able to use language files to display text in the user's language. So far, the only additional language other than English is German. If you want to create another language file, download the German .po file (which also contains the English translations), and replace all the "msgstr" entries (there aren't too many). Then send me a pull request or simply post the file to the issue tracker (if you don't know how to git).

## Contributing

Contributions are welcome! Feel free to fork this repository and submit pull requests.

## License

This project is licensed under the GPLv3 License. See the COPYING file for details.

## Author

The Tor-Router Debian package was created by m4dm4x1337.

## Donations 🥺

 ❤️ Please donate ❤️

![QR code for donations](https://raw.githubusercontent.com/m4dm4x1337/tor-router-gnome/master/tor-router-gnome/usr/share/pixmaps/tor-router-gnome-donation.png)

This project is open source, and the only income comes from the donators. If you like the project, please donate, thank you!

[bitcoin:bc1q9ha0l0tt7dghcpgext8jppejandefeshcukpxx](bitcoin:bc1q9ha0l0tt7dghcpgext8jppejandefeshcukpxx)
