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


