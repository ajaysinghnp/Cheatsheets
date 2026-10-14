---
title: Managing Network Configuration Guide
slug: managing-network-and-configurations
author: ajay-singh
excerpt: Linux commands to manage network and configurations
tags: ["Linux", "Network", "SysAdmin"]
created_at: 2023-05-01T10:00:00Z
updated_at: 2023-05-02T12:00:00Z
---

# Linux Network Configuration Guide

This markdown note covers essential concepts, tools, and configuration steps for managing networks on Linux.

## Key Concepts

1. NIC (Network Interface Controller): Physical/virtual network device (e.g., eth0, wlan0).

2. IP Address: Unique identifier for a device on a network (IPv4/IPv6).

3. Subnet Mask: Defines the network segment (e.g., 255.255.255.0 = /24).

4. Gateway: Router interface connecting to external networks.

5. DNS: Translates domain names to IP addresses (e.g., 8.8.8.8).

## Configuration Files & Locations

| Purpose           | File/Directory                  | Description                             |
| :---------------- | :------------------------------ | :-------------------------------------- |
| NIC Configuration | /etc/network/interfaces         | Legacy config for Debian-based systems. |
|                   | /etc/netplan/*.yaml             | Modern config (Ubuntu 18.04+).          |
|                   | /etc/sysconfig/network-scripts/ | RHEL/CentOS/Fedora.                     |
| DNS Resolution    | /etc/resolv.conf                | DNS servers (often auto-generated).     |
| Static Hostnames  | /etc/hosts                      | Local hostname-to-IP mappings.          |
| Hostname          | /etc/hostname                   | System hostname.                        |

## Essential Command-Line Tools

| Tool        | Command Examples                     | Purpose                               |
| :---------- | :----------------------------------- | :------------------------------------ |
| ip          | ip addr show                         | Show IP addresses.                    |
|             | ip route add default via 192.168.1.1 | Add default gateway.                  |
| nmcli       | nmcli connection show                | List NetworkManager connections.      |
|             | nmcli device wifi list               | Scan Wi-Fi networks.                  |
| ifconfig    | ifconfig eth0                        | Legacy interface config (deprecated). |
| ping        | ping google.com                      | Test network connectivity.            |
| dig         | dig example.com                      | DNS lookup.                           |
| ss/netstat  | ss -tuln                             | Show open ports/services.             |
| hostnamectl | hostnamectl set-hostname myserver    | Change hostname.                      |

## Common Configuration Examples

**Static IP via `/etc/netplan/*.yaml` (Ubuntu)**

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0: # check for the name of your adapter here
      dhcp4: no # disable the dhcp
      addresses:
        - 192.168.1.10/24 # check for your static IP address for your server
      routes:
        - to: default
          via: 192.168.1.1  # check for your gateway address
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1] # change this value to your DNS Servers
```

**Apply:**

```bash
sudo netplan apply
```

### Static IP via `/etc/network/interfaces` (Debian)

```bash
auto eth0
iface eth0 inet static
  address 192.168.1.10
  netmask 255.255.255.0
  gateway 192.168.1.1
  dns-nameservers 8.8.8.8
```

Apply:

```bash
sudo systemctl restart networking
```

### DHCP Configuration

```bash
auto eth0
iface eth0 inet dhcp  # For /etc/network/interfaces
```

Or via `nmcli`:

```bash
nmcli con mod "ConnectionName" ipv4.method auto  # Enable DHCP
```

### DNS Configuration

**Manually edit `/etc/resolv.conf` (temporary):**

```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```

**Persistent DNS (with Netplan):**

```bash
nameservers:
  addresses: [8.8.8.8, 8.8.4.4]
```

### Troubleshooting Commands

```bash
ip addr show          # Check interfaces & IPs
ip route              # View routing table
ping 8.8.8.8          # Test internet connectivity
dig google.com        # Verify DNS resolution
ss -tuln              # List active ports
journalctl -u systemd-networkd  # View network service logs
```

### NetworkManager Tips

- Enable/disable a connection:

  ```bash
  nmcli connection up "MyEthernet"
  nmcli connection down "MyEthernet"
  ```

- Add a Wi-Fi connection:

  ```bash
  nmcli device wifi connect "SSID" password "PASSWORD"
  ```

### Advanced Tools

- `tcpdump`: Packet sniffing.

- `iptables/nftables`: Firewall configuration.

- `wireshark`: GUI packet analysis.

- `bridge-utils`: Manage bridge interfaces.

> **Note**: Always backup config files before editing! Use `sudo` for administrative tasks.

Save this as `linux_network_config.md` and expand sections as needed for your environment!
