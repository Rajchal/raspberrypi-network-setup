# Raspberrypi Network Setup

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

## Verify dnsmasq Config

## Restart dnsmasq

## keep reviewing logs using 
```bash
sudo journalctl -veu dnsmasq.service
```

> [!IMPORTANT]
> use iwconfig to see if the network is in managed state or monitor state

```bash
sudo apt install wireless-tools
```

Check wlan0 status

```bash
iwconfig wlan0
```

## Check journal of hostapd aswell there can be problem there

```bash
sudo apt update
sudo apt install net-tools
```

install net-tools for usinf ifconfig to bring up or down the wlan0

```bash
# example
sudo ifconfig wlan0 down
```

you can also use ip command

```bash
sudo ip link set wlan0 down
sudo ip link set wlan0 up
```

## Rebooting also helps sometimes
```bash
sudo reboot
```

## Create PID dir 
```bash
sudo mkdir -p /run/hostapd
sudo chown root:root /run/hostapd
sudo chmod 755 /run/hostapd
```

## Stop conflicting services

```bash
sudo systemctl stop wpa_supplicant
sudo systemctl disable wpa_supplicant
sudo systemctl stop NetworkManager
sudo systemctl disable NetworkManager
```

## This is what worked for me
- Since i was getting error that my firmware is rejecting country settings 
- remember this error is only visible in raspberry pi tty not in ssh connection
- i realised it only after looking at it and not in the ssh terminal

```bash
sudo iw reg set 00
```
this sets to global which will work now

> Add iw package if it doesnt exist
> ```bash
> sudo iw reg get
> ```

## Disabling network dispatcher also helped

```bash
sudo systemctl stop networkd-dispatcher
sudo systemctl disable networkd-dispatcher
```

## Stop and disable `systemd-resolved`
```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

also mask it bro it really helps

```bash
sudo systemctl mask systemd-resolved
```

## By doing all this once reboot your system 

```bash
sudo reboot
```

## Hopefully this should work notify me if you get better methods in `rajchalanjal1@gmail.com`


