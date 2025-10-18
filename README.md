# Home VPN with Raspberry Pi 5 + WireGuard (PiVPN)

## Overview
A guide to setting up a secure home VPN using Raspberry Pi 5, WireGuard (PiVPN) with DuckDNS, UFW, and Fail2Ban, complete with setup scripts, documentation, and diagrams.

## System Diagram

[![Raspberry Pi WireGuard VPN Diagram](images/system-diagram.png)](images/system-diagram.png)

## Quick Setup

### Flash OS with Raspberry Pi Imager
- Device: Raspberry Pi 5
- OS: Raspberry Pi OS Lite (64-bit)
- Advanced options:
  - Hostname: raspberrypi (change hostname or keep default)
  - Enable SSH (key-only)
  - Username: pi
  - Password: your_password
  - (Optional) Configure Wi-Fi
- Write and verify image, then eject SD card.

### SSH & Update
```bash
ssh pi@<hostname>.local
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

### Secure SSH
```bash
ssh-keygen -t ed25519
ssh-copy-id pi@<hostname>.local
sudo nano /etc/ssh/sshd_config
# set PasswordAuthentication no
sudo systemctl restart ssh
```

### Security Tools & Firewall
```bash
sudo apt install unattended-upgrades fail2ban ufw -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
sudo ufw allow 22/tcp
sudo ufw allow 51820/udp
sudo ufw enable
```

### Static IP
```bash
sudo nano /etc/dhcpcd.conf
# interface eth0
# static ip_address=192.168.1.10/24
# static routers=192.168.1.1
# static domain_name_servers=1.1.1.1 8.8.8.8
sudo reboot
```

### WireGuard via PiVPN
```bash
curl -L https://install.pivpn.io | bash
# Choose WireGuard, port 51820/UDP, DNS Cloudflare, enable unattended upgrades
```

### DuckDNS Dynamic DNS
```bash
curl "https://www.duckdns.org/update?domains=SUBDOMAIN&token=YOURTOKEN&ip="
crontab -e
# */5 * * * * /usr/bin/curl -s "https://www.duckdns.org/update?domains=SUBDOMAIN&token=YOURTOKEN&ip=" >/dev/null 2>&1
```

### Router Port Forwarding
Forward UDP 51820 → 192.168.1.10:51820

### Add Client Profiles
```bash
pivpn add
pivpn -qr
```

### Verify Connection
Check WireGuard status 
```bash
sudo wg show
```
Visit [whatismyipaddress.com](https://whatismyipaddress.com) to confirm your VPN IP.

## Troubleshooting
- If the SD card fails to write, erase and re-flash it using Raspberry Pi Imager’s 'Erase' option.
- If SSH via IP doesn’t work, use the hostname instead: ssh pi@<hostname>.local.

## Resources
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- [PiVPN Installer](https://pivpn.io)
- [WireGuard Docs](https://www.wireguard.com/)
- [DuckDNS](https://www.duckdns.org/)