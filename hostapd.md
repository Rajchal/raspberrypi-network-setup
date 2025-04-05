# Hostapd setup

1. Ensure the wireless interface is up
```bash
sudo ip link set wlan0 up
```
2. Check Wireless Interface Status
```bash
sudo iwconfig wlan0
```

3. Restart hostapd
```bash
sudo systemctl restart hostapd
```

4. Disable wpa\_supplicant
```bash
sudo systemctl stop wpa_supplicant
sudo systemctl disable wpa_supplicant
```

5. Verify IP forwarding
```bash
sudo sysctl net.ipv4.ip_forward
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -p
```

6. Check IP table rules
```bash
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F
sudo iptables -t nat -X
sudo iptables -t mangle -F
sudo iptables -t mangle -X
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```

7. Ensure correct interface in dhcpcd.conf
```bash
sudo nano /etc/dhcpcd.conf
```

- make sure there is:
```bash
interface wlan0
static ip_address=192.168.4.1/24
static routers=192.168.4.1
static domain_name_servers=8.8.8.8 8.8.4.4
```

- create resolv.conf
```bash
sudo mkdir -p /run/dnsmasq
sudo nano /run/dnsmasq/resolv.conf
```

- add:
```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```

- add 
```bash
sudo nano /etc/dnsmasq.conf
# resolv-file=/run/dnsmasq/resolv.conf
```

- edit
```bash
sudo vim /etc/dhcpcd.conf
```

- to
```bash
# A sample configuration for dhcpcd.
# See dhcpcd.conf(5) for details.

# Allow users of this group to interact with dhcpcd via the control socket.
#controlgroup wheel

# Inform the DHCP server of our hostname for DDNS.
#hostname

# Use the hardware address of the interface for the Client ID.
#clientid
# or
# Use the same DUID + IAID as set in DHCPv6 for DHCPv4 ClientID as per RFC4361.
# Some non-RFC compliant DHCP servers do not reply with this set.
# In this case, comment out duid and enable clientid above.
duid

# Persist interface configuration when dhcpcd exits.
persistent

# vendorclassid is set to blank to avoid sending the default of
# dhcpcd-<version>:<os>:<machine>:<platform>
vendorclassid

# A list of options to request from the DHCP server.
option domain_name_servers, domain_name, domain_search
option classless_static_routes
# Respect the network MTU. This is applied to DHCP routes.
option interface_mtu

# Request a hostname from the network
option host_name

# Most distributions have NTP support.
#option ntp_servers

# Rapid commit support.
# Safe to enable by default because it requires the equivalent option set
# on the server to actually work.
option rapid_commit

# A ServerID is required by RFC2131.
require dhcp_server_identifier

# Generate SLAAC address using the Hardware Address of the interface
#slaac hwaddr
# OR generate Stable Private IPv6 Addresses based from the DUID
slaac private

interface wlan0
static ip_address=192.168.4.1/24
static routers=192.168.4.1
static domain_name_servers=8.8.8.8 8.8.4.4
ipv6rs=false
```

- make sure
```bash
sudo apt install dhcpcd
sudo systemctl start dhcpcd
sudo systemctl enable dhcpcd
```

## Disable netplan
```bash
sudo rm /etc/netplan/50-cloud-init.yaml
sudo nano /etc/netplan/01-netcfg.yaml
```

- add this:

```bash
network:
  version: 2
  renderer: networkd
```

# Reboot and this should work
```bash
sudo reboot
```



