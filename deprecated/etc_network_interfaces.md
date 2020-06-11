# Deprecated way of configuring Ubuntu's network interface

> Using `/etc/network/interfaces` was deprecated since Ubuntu 18.04. Use
`/etc/netplan/*.yml` instead.

Create or edit your network configuration
`/etc/network/interfaces` like this. You probably need to use `sudo` before your
editor command in order to save the changes.
```sh
# In VirtualBox's VM window.
sudo vim /etc/network/interfaces
```
```sh
# /etc/network/interfaces

# Show all interfaces: ls /sys/class/net

# The loopback network interface.
auto lo
iface lo inet loopback

# Adapter 1: the primary network interface (NAT)
# We request from a DHCP server (provided by VirtualBox) for a dynamic IP address.
auto enp0s3
iface enp0s3 inet dhcp

# Adapter 2: the host-only network interface (Host-Only).
# We use a static IP.
auto enp0s8
iface enp0s8 inet static
# These numbers agree with the network adapter you created in
# VirtualBox File > Host Network Manager.
address 192.168.56.101
netmask 255.255.255.0
```

■
