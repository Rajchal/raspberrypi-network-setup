# raspberrypi-network-setup

This is quick guide to network setup for raspberrypi and mistakes to not make


1. Update Your Pi
```bash
sudo apt update
sudo apt upgrade
```

2. Install necessary software
```bash
sudo apt install hostapd dnsmasq
sudo systemctl stop hostapd
sudo systemctl stop dnsmasq
```

3. Configure static IP in `dhcpd.conf`
```bash
sudo nano /etc/dhcpcd.conf
```

add this

```bash
interface wlan0
static ip_address=192.168.4.1/24
nohook wpa_supplicant
```

4. Configure the DHCP server
```bash
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
sudo nano /etc/dnsmasq.conf
```

and the following config

```bash
interface=wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
```

5. Configure the access point software
```bash
sudo nano /etc/hostapd/hostapd.conf
```

add the following line to config

```bash
interface=wlan0
driver=nl80211
ssid=Your_SSID
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=Your_Passphrase
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

point hostapd to config file
```bash
sudo nano /etc/default/hostapd
```

un comment

```bash
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

6. Enable IP routing
```bash
sudo nano /etc/sysctl.conf
```

un comment

```bash
net.ipv4.ip_forward=1
```

apply the change

```bash
sudo sysctl -p
```

7. Configure NAT between wlan0 and eth0
```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

edit the `rc.local`

```bash
sudo nano /etc/rc.local
```

add the following line just above exit 0

```bash
iptables-restore < /etc/iptables.ipv4.nat
```

8. Start the services
```bash
sudo systemctl start hostapd
sudo systemctl start dnsmasq
sudo systemctl enable hostapd
sudo systemctl enable dnsmasq
```

9. Enable SSH access
```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

## So upto this point you are consider to make the thing working but it wont so after this ima tell you how to fix

Install Necessary Softwartes
```bash
sudo apt install hostapd dnsmasq netfilter-persistent iptables-persistent -y
sudo systemctl stop hostapd
sudo systemctl stop dnsmasq
```

Config Static IP
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

add the following

```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
  wifis:
    wlan0:
      dhcp4: no
      addresses:
        - 192.168.4.1/24
      access-points:
        "Your_SSID":
          password: "Your_Passphrase"
```

apply 
```bash
sudo netplan apply
```

If you get masking error
```bash
sudo systemctl unmask hostapd
```

start hostapd
```bash
sudo systemctl start hostapd
sudo systemctl enable hostapd
```

restart dnsmasq
```bash
sudo systemctl restart dnsmasq
sudo systemctl enable dnsmasq
```

check status
```bash
sudo systemctl status dnsmasq
```
## Mostly you come to port conflits in this method you can stop this procces which happens in :67 and :53

1. Check for services using port 67 and 53
```bash
sudo lsof -i :67
sudo lsof -i :53
```

2. Stop any conflicting service example if `dhcp`
```bash
sudo systemctl stop dhcpcd
```


