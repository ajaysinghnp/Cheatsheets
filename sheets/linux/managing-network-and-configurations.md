---

title: Managing Network Configuration Guide
slug: managing-network-and-configurations
author: Ajay Singh
excerpt: A practical Linux networking reference covering interfaces, IP addressing, routing, DNS, DHCP, IPv6, firewalls, network configuration, and troubleshooting.
tags: ["Linux", "Network", "SysAdmin"]
created_at: 2025-07-11T05:00:04Z
---

This cheatsheet covers the essential commands, configuration files, and troubleshooting techniques for managing network connectivity on Linux servers and desktops.

Examples use common Linux networking tools. Interface names, connection profiles, configuration paths, and available commands may differ by distribution.

## 1. Identify Your Network Environment

Before changing network settings, determine which interfaces and network management services are in use.

### Check the Linux distribution

```bash
cat /etc/os-release
uname -r
```

### Identify network interfaces

```bash
ip link show
ip -br link
ip -br addr
```

### Check the default gateway and routing table

```bash
ip route
ip route get 8.8.8.8
ip -6 route
```

### Identify the network manager

```bash
systemctl is-active NetworkManager
systemctl is-active systemd-networkd
systemctl is-active networking
```

Check whether NetworkManager or systemd-networkd manages a particular interface:

```bash
nmcli device status
networkctl list
```

These services may coexist. Determine which service actually manages the interface before editing its configuration.

## 2. Essential Network Concepts

| Concept         | Description                                                | Example             |
| :-------------- | :--------------------------------------------------------- | :------------------ |
| NIC             | Physical or virtual network interface                      | `enp1s0`, `eth0`    |
| IPv4 address    | Device address within an IPv4 network                      | `192.168.1.10`      |
| CIDR prefix     | Defines the network size                                   | `/24`               |
| Default gateway | Router used for destinations without a more specific route | `192.168.1.1`       |
| DNS resolver    | Resolves hostnames to IP addresses                         | `192.168.1.1`       |
| DHCP            | Automatically assigns network configuration                | DHCP lease          |
| MAC address     | Link-layer hardware or interface address                   | `02:11:22:33:44:55` |
| VLAN            | Separates Layer 2 network segments                         | VLAN 20             |
| MTU             | Maximum size of a network-layer packet supported on a link | `1500`              |

### Useful CIDR examples

| CIDR | Subnet mask | Total IPv4 addresses |
| :-- | :-- | --: |
| `/8` | `255.0.0.0` | 16,777,216 |
| `/16` | `255.255.0.0` | 65,536 |
| `/24` | `255.255.255.0` | 256 |
| `/25` | `255.255.255.128` | 128 |
| `/30` | `255.255.255.252` | 4 |

For ordinary IPv4 subnets, network and broadcast addresses are generally not assigned to hosts. `/31` and `/32` have special uses.

## 3. Essential Network Commands

### IP addresses and interfaces

```bash
ip addr show
ip -br addr
ip link show
ip link show dev enp1s0

# Temporarily enable or disable an interface
sudo ip link set enp1s0 up
sudo ip link set enp1s0 down

# Add or remove an address at runtime
sudo ip addr add 192.168.1.10/24 dev enp1s0
sudo ip addr del 192.168.1.10/24 dev enp1s0
```

Changes made using `ip` are generally runtime-only and may disappear after a reboot or network service restart.

### Routing

```bash
# View IPv4 and IPv6 routes
ip route show
ip -6 route show

# Inspect the route selected for a destination
ip route get 8.8.8.8

# Add a temporary default route
sudo ip route add default via 192.168.1.1 dev enp1s0

# Add a route to another subnet
sudo ip route add 10.20.40.0/24 via 192.168.1.254

# Remove a route
sudo ip route del 10.20.40.0/24
```

Do not add a second default route blindly. Multiple default routes may be intentional, but route metrics and policy routing determine which path is selected.

### Socket and port inspection

```bash
# Listening TCP and UDP sockets
sudo ss -tuln

# Include process information
sudo ss -tulpn

# Established TCP connections
ss -tn state established

# Inspect a particular port
sudo ss -tulpn 'sport = :443'
```

Common options:

- `-t`: TCP
- `-u`: UDP
- `-l`: Listening sockets
- `-n`: Numeric addresses and ports
- `-p`: Process information

### Interface statistics

```bash
ip -s link show
cat /proc/net/dev
```

Look for increasing error or drop counters when investigating connectivity problems.

## 4. Network Configuration Files

| Purpose | Configuration location | Typical use |
| :-- | :-- | :-- |
| Netplan | `/etc/netplan/*.yaml` | Ubuntu and other systems using Netplan |
| NetworkManager profiles | `/etc/NetworkManager/system-connections/` | NetworkManager-managed systems |
| Debian interfaces | `/etc/network/interfaces` | Systems using ifupdown |
| systemd-networkd | `/etc/systemd/network/*.network` | systemd-networkd-managed interfaces |
| DNS resolver | `/etc/resolv.conf` | Resolver configuration or a managed symlink |
| Local hostname mappings | `/etc/hosts` | Static hostname-to-address mappings |
| System hostname | `/etc/hostname` | Persistent system hostname |
| Hosts and network databases | `/etc/nsswitch.conf` | Name-service lookup order |

**Important:** Do not configure the same interface independently through multiple network managers. On systems using a managed `/etc/resolv.conf`, direct edits may be overwritten.

## 5. Configure a Static IP with Netplan

Netplan is commonly used on Ubuntu systems. It generates configuration for a backend such as `systemd-networkd` or NetworkManager.

### Example configuration

First identify the interface:

```bash
ip -br addr
ip route
ls -l /etc/netplan/
```

Back up the existing configuration before editing:

```bash
sudo cp -a /etc/netplan "/etc/netplan.backup-$(date +%F-%H%M%S)"
sudo nano /etc/netplan/01-network.yaml
```

Example static IPv4 configuration:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: false
      addresses:
        - 192.168.1.10/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 192.168.1.1
          - 1.1.1.1
```

Replace `enp1s0`, the IP address, prefix, gateway, and DNS servers with values appropriate for your network.

If NetworkManager manages the interface, use `renderer: NetworkManager` where appropriate. Avoid creating a competing profile for an already-managed connection.

### Validate and apply

```bash
sudo netplan generate
sudo netplan try
```

`netplan try` attempts the configuration and allows you to confirm or roll it back if connectivity is lost. Read its prompts carefully, especially when configuring a remote server.

After confirming the configuration:

```bash
ip -br addr
ip route
resolvectl status
```

Use `sudo netplan apply` when you are ready to apply the configuration without the interactive rollback workflow.

## 6. Configure Networking with NetworkManager

NetworkManager is common on desktop Linux and many server installations.

### List connections and devices

```bash
nmcli connection show
nmcli device status
nmcli device show
nmcli connection show --active
```

### Configure a static IPv4 address

First find the actual connection profile name:

```bash
nmcli connection show
```

Modify the existing profile:

```bash
sudo nmcli connection modify "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 192.168.1.10/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "192.168.1.1 1.1.1.1"
```

Activate the modified connection:

```bash
sudo nmcli connection up "Wired connection 1"
```

Reactivating a connection may interrupt your session. Use console or out-of-band access when changing the network interface carrying your SSH connection.

### Switch back to DHCP

```bash
sudo nmcli connection modify "Wired connection 1" \
  ipv4.method auto \
  ipv4.addresses "" \
  ipv4.gateway "" \
  ipv4.dns ""

sudo nmcli connection up "Wired connection 1"
```

### Manage Wi-Fi

```bash
# List available Wi-Fi networks
nmcli device wifi list

# Connect to a Wi-Fi network
nmcli device wifi connect "SSID" --ask

# List saved connection profiles
nmcli connection show

# Activate or deactivate a profile
sudo nmcli connection up "ProfileName"
sudo nmcli connection down "ProfileName"
```

Using `--ask` prompts for the Wi-Fi password instead of placing it directly in your shell command.

## 7. Configure Networking with Debian ifupdown

On systems that use ifupdown, the primary configuration is often `/etc/network/interfaces`.

### Static IPv4 example

```bash
sudo nano /etc/network/interfaces
```

Example:

```text
auto lo
iface lo inet loopback

auto enp1s0
iface enp1s0 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
```

DNS configuration depends on the resolver setup. Some systems use `/etc/resolv.conf`, while others use a resolver service.

### DHCP example

```text
auto enp1s0
iface enp1s0 inet dhcp
```

### Apply changes

On a suitable system using ifupdown:

```bash
sudo ifdown enp1s0
sudo ifup enp1s0
```

These commands can disconnect a remote session and may fail if the interface is not managed by ifupdown. Check your distribution's service and recovery procedures before applying changes remotely.

## 8. DHCP Configuration and Troubleshooting

DHCP automatically supplies settings such as the IP address, gateway, and DNS servers.

### Inspect the current configuration

```bash
ip -br addr
ip route
nmcli device show
```

### Renew a DHCP lease

For a NetworkManager-managed connection:

```bash
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

For a system using a dedicated DHCP client, the renewal command depends on the client and distribution.

Avoid running multiple DHCP clients against the same interface.

### Common DHCP issues

- No address or an unexpected `169.254.x.x` IPv4 address.
- No default route.
- Duplicate IP address warnings.
- DHCP server unreachable because of VLAN, switch, or Wi-Fi configuration.
- Lease obtained, but DNS or internet connectivity still fails.

## 9. DNS Configuration and Diagnostics

### Inspect resolver configuration

```bash
cat /etc/resolv.conf
resolvectl status
resolvectl query example.com
```

`resolvectl` is available on systems using `systemd-resolved`.

### Test DNS lookups

```bash
# Basic lookup
getent hosts example.com

# Query the configured DNS resolver
dig example.com

# Query a specific resolver
dig @1.1.1.1 example.com

# Query record types
dig A example.com
dig AAAA example.com
dig MX example.com
dig TXT example.com

# Reverse DNS lookup
dig -x 192.168.1.10

# Short output
dig +short example.com
```

### Distinguish DNS failure from network failure

```bash
# Test IP connectivity
ping -c 4 1.1.1.1

# Test DNS resolution
getent hosts example.com

# Test HTTPS connectivity
curl -I --max-time 10 https://example.com
```

Interpret the results carefully:

- If IP connectivity works but hostname resolution fails, investigate DNS.
- If DNS works but HTTPS fails, investigate routing, firewall rules, TLS, proxies, or the destination service.
- A failed ping alone does not prove that a host is offline; ICMP may be blocked.

### Configure persistent DNS

Prefer your active network manager's DNS configuration, such as the Netplan or NetworkManager examples above.

Editing `/etc/resolv.conf` directly is generally temporary when that file is generated or managed by another service.

## 10. Hostnames and Local Name Resolution

### View or change the hostname

```bash
hostname
hostnamectl
```

Set a persistent hostname:

```bash
sudo hostnamectl set-hostname myserver
```

Verify:

```bash
hostnamectl status
```

### Edit local hostname mappings

```bash
sudo nano /etc/hosts
```

Example:

```text
127.0.0.1   localhost
127.0.1.1   myserver
192.168.1.20 nas.local
```

Use `/etc/hosts` for local static mappings. For larger networks, centralized DNS is usually easier to maintain.

## 11. IPv6 Essentials

### Inspect IPv6 addresses and routes

```bash
ip -6 addr show
ip -6 route show
ip -6 neigh show
```

### Test IPv6 connectivity

```bash
ping -6 -c 4 2606:4700:4700::1111
curl -6 -I --max-time 10 https://example.com
```

### IPv6 troubleshooting checklist

- Verify that the interface has the expected IPv6 address.
- Check the IPv6 default route.
- Check router advertisements and DHCPv6 if applicable.
- Verify IPv6 DNS records and resolver behavior.
- Check firewall rules for IPv6 separately from IPv4.
- Verify that the upstream network actually provides IPv6 connectivity.

Do not disable IPv6 as a first troubleshooting step. Some systems and applications depend on it.

## 12. Firewall and Open Port Diagnostics

### UFW

UFW is commonly used as a simplified firewall interface on Ubuntu.

```bash
sudo ufw status verbose
sudo ufw status numbered

# Allow SSH before enabling UFW on a remote server
sudo ufw allow OpenSSH

# Allow HTTPS
sudo ufw allow 443/tcp

# Enable the firewall
sudo ufw enable
```

Before enabling or changing firewall rules remotely, confirm that your SSH port is allowed and that you have a recovery path.

### nftables

```bash
sudo nft list ruleset
sudo nft list tables
```

### iptables compatibility commands

```bash
sudo iptables -L -n -v
sudo iptables -t nat -L -n -v
sudo ip6tables -L -n -v
```

Some systems use nftables underneath the iptables compatibility interface. Use the firewall manager actually responsible for the active rules rather than modifying several systems independently.

### Check whether a service is listening

```bash
sudo ss -tulpn
sudo ss -tulpn 'sport = :22'
```

A service listening on `127.0.0.1` may not accept connections through the server's LAN address. Check the listening address as well as the firewall.

## 13. Packet Capture and Network Inspection

### Capture packets with tcpdump

```bash
# Capture traffic on an interface
sudo tcpdump -i enp1s0

# Capture traffic involving a host
sudo tcpdump -i enp1s0 host 192.168.1.20

# Capture DNS traffic
sudo tcpdump -i enp1s0 port 53

# Capture HTTPS traffic
sudo tcpdump -i enp1s0 port 443

# Write a capture to a file
sudo tcpdump -i enp1s0 -w network-capture.pcap

# Read a saved capture
sudo tcpdump -r network-capture.pcap
```

Capture files may contain sensitive network metadata. Store them securely and avoid sharing them without reviewing their contents.

### Trace the network path

```bash
traceroute example.com
tracepath example.com
```

If `traceroute` is unavailable, install it using your distribution's package manager. Some networks block the probe types used by tracing tools, so missing hops do not necessarily indicate a failure.

### Check link speed and negotiated settings

```bash
sudo ethtool enp1s0
```

`ethtool` is useful for diagnosing link state, speed, duplex, and interface negotiation on supported Ethernet devices.

## 14. VLANs and Network Interfaces

### Inspect interface details

```bash
ip -d link show
ip link show
ip -s link show
```

### Create a temporary VLAN interface

For a VLAN-aware Linux host, the following example creates VLAN 20 on `enp1s0`:

```bash
sudo ip link add link enp1s0 name enp1s0.20 type vlan id 20
sudo ip link set enp1s0.20 up
sudo ip addr add 192.168.20.10/24 dev enp1s0.20
```

Remove the temporary VLAN interface:

```bash
sudo ip link delete enp1s0.20
```

These commands create a runtime configuration only. The switch port must also be configured appropriately to carry the VLAN tag, and persistent VLAN configuration should be managed through Netplan, NetworkManager, or the applicable network manager.

## 15. Network Service Logs and Diagnostics

### systemd-networkd

```bash
systemctl status systemd-networkd
journalctl -u systemd-networkd -b
journalctl -u systemd-networkd --since "30 minutes ago"
```

### NetworkManager

```bash
systemctl status NetworkManager
journalctl -u NetworkManager -b
nmcli general status
```

### systemd-resolved

```bash
systemctl status systemd-resolved
journalctl -u systemd-resolved -b
```

### General system and kernel logs

```bash
journalctl -b -p warning
dmesg --level=err,warn
```

Use the service relevant to your system. A missing service or empty log is not necessarily an error if the distribution uses another network manager.

## 16. Common Troubleshooting Workflow

Follow this sequence when a Linux system loses network connectivity.

### Step 1: Check the physical or virtual link

```bash
ip -br link
ip -s link
```

Confirm that the interface is up and that the expected link is present.

### Step 2: Check the IP address

```bash
ip -br addr
```

Verify the assigned address, prefix, and whether the interface has the expected IPv4 or IPv6 configuration.

### Step 3: Check the routing table

```bash
ip route
ip route get 1.1.1.1
```

Verify that a valid route exists to the destination and that the selected interface and gateway are correct.

### Step 4: Test the local gateway

```bash
ping -c 4 192.168.1.1
ip neigh show
```

Replace the example gateway with your actual router address. If the gateway does not respond, check the local link, VLAN, address, subnet, and router configuration. Remember that ICMP may be blocked.

### Step 5: Test external IP connectivity

```bash
ping -c 4 1.1.1.1
```

If the gateway works but external connectivity fails, investigate the default route, upstream router, firewall, and internet connection.

### Step 6: Test DNS resolution

```bash
getent hosts example.com
dig example.com
```

If IP connectivity works but DNS fails, inspect the configured resolver and any DNS filtering or forwarding services.

### Step 7: Test the application port

```bash
curl -v --connect-timeout 5 --max-time 10 https://example.com
```

For a local service, check the listening socket, service status, firewall, and whether the service is bound to the expected address.

### Step 8: Inspect logs

```bash
journalctl -b -p warning
```

Then inspect the logs for the relevant network manager or application.

## 17. Quick Reference

| Task | Command |
| :-- | :-- |
| Show IP addresses | `ip addr` |
| Compact interface status | `ip -br addr` |
| Show routing table | `ip route` |
| Inspect a selected route | `ip route get 1.1.1.1` |
| Show listening ports | `sudo ss -tulpn` |
| Test connectivity | `ping -c 4 1.1.1.1` |
| Resolve a hostname | `getent hosts example.com` |
| Inspect DNS | `resolvectl status` |
| Query DNS | `dig example.com` |
| Check NetworkManager | `nmcli device status` |
| Check systemd-networkd | `networkctl list` |
| Check firewall | `sudo ufw status verbose` |
| Inspect nftables | `sudo nft list ruleset` |
| Capture network packets | `sudo tcpdump -i enp1s0` |
| Trace the network path | `traceroute example.com` |
| Inspect interface errors | `ip -s link` |
| View recent system warnings | `journalctl -b -p warning` |

## 18. Best Practices

- Back up configuration files before making changes.
- Identify the active network manager before editing configuration.
- Use persistent configuration through the appropriate network manager rather than relying on temporary `ip` commands.
- Keep a console or out-of-band recovery option when changing a remote server's network settings.
- Verify the IP address, routing table, DNS, and application connectivity after every significant change.
- Avoid exposing unnecessary listening services or opening firewall ports without a clear requirement.
- Keep IPv4 and IPv6 firewall behavior in mind.
- Use least-privilege access and `sudo` only when necessary.
- Record the original configuration and the steps needed to roll back a change.
