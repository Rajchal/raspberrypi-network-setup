# dnsmasq setup

## This setup is really important for the pi since you are not relying on dhcp

### First ping a public ip to test
```bash
ping 8.8.8.8
```

### Also check dhcp configuration for eth0 since it will reply on it
```bash
cat /etc/network/interfaces
```

### Should show
> auto eth0
> iface eth0 intet dhcp

### Check DNS Configuration
```bash
cat /etc/resolv.conf
```

- Should contain `nameserver` entries such as:
> nameserver 8.8.8.8
> nameserver 8.8.4.4

## From here i have a step by step checlist:

1. Enable the `networking` services
```bash
sudo systemctl enable networking.service
```

2. Configure `/etc/network/interfaces` for DHCP
```bash
sudo nano /etc/network/interfaces
```

## Disabling some stuffs to start dnsmasq
```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

- you can even mask it
```bash
sudo systemctl mask systemd-resolved
```

- configure dnsmasq
```bash
sudo nano /etc/dnsmasq.conf
```
interface=wlan0  # Replace with your wireless interface
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
dhcp-option=3,192.168.4.1
dhcp-option=6,8.8.8.8,8.8.4.4
listen-address=127.0.0.1
server=8.8.8.8
server=8.8.4.4
```

- restart the dnsmasq
```bash
sudo systemctl restart dnsmasq
```

- check status
```bash
sudo systemctl status dnsmasq
```

- Examine system logs
```bash
sudo journalctl -b | grep dnsmasq
```
- Remove `resolvconf`
```bash
sudo apt remove --purge resolvconf
```

- prevent resolve.conf from overwriting
```bash
sudo chattr +i /etc/resolv.conf
```

- mask resolvconf
```bash
sudo systemctl mask resolvconf.service
sudo systemctl mask resolvconf.socket
```

> [!IMPORTANT]
> - You have to do this !!!
> ```bash
> sudo nano /lib/systemd/system/dnsmasq.service
> ```
> - Look for any lines that call resolvconf and comment them out. For example:
> `#ExecStartPost=/usr/sbin/resolvconf -u`

- reload systemd
```bash
sudo systemctl daemon-reload
```

- restart dnsmasq

## Check firewall
```bash
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT
sudo iptables -A OUTPUT -p udp --sport 53 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 53 -j ACCEPT
sudo ufw allow 53
```
# This should make dnsmasq work still improvements can be done in this method as it is brute force

