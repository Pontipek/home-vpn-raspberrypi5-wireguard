# Home VPN with Raspberry Pi 5 + WireGuard (PiVPN)

## Overview
A guide to setting up a secure home VPN using Raspberry Pi 5, WireGuard (PiVPN) with DuckDNS, UFW, and Fail2Ban, complete with setup scripts, documentation, and diagrams.

## System Diagram

[![Raspberry Pi WireGuard VPN Diagram](images/system-diagram.png)](images/system-diagram.png)

## Quick Setup

### Flash OS with Raspberry Pi Imager
- **Device**: Raspberry Pi 5
- **OS**: Raspberry Pi OS Lite (64-bit)
- **Advanced options**:
  - Hostname: pivpn (change hostname or keep default)
  - Enable SSH (key-only)
  - Username: pi (change username or keep default)
  - Password: your_password
  - (Optional) Configure Wi-Fi
- Write and verify image, then eject SD card.

### Boot, connect to pi via SSH & update packages
Insert the SD card, power on the Pi, and connect via Ethernet.  
From your terminal (Windows or Git Bash):
```bash
ssh <username>@<hostname>.local
sudo apt update && sudo apt full-upgrade -y && sudo reboot
```

### Secure SSH Access
Generate a key pair and disable password logins (run these **on your compter**):
```bash
ssh-keygen -t ed25519
ssh-copy-id <username>@<hostname>.local
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
# set PasswordAuthentication no
sudo systemctl restart ssh
```

### Install Security Tools
```bash
sudo apt install unattended-upgrades fail2ban -y
sudo systemctl enable --now fail2ban
sudo systemctl status fail2ban --no-pager
```

### Assign Static IP
**Via terminal**

Edit network configuration:
```bash
sudo nano /etc/dhcpcd.conf
```
Add:
```
interface eth0
static ip_address=192.168.0.10/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.10
```
Reboot the Pi:
```bash
sudo reboot
```
**Via router**
[![Assign Static PI via Router](images/address-reservation.png)](images/address-reservation.png)


### Install WireGuard via PiVPN & Enable NAT and Routing
```bash
curl -L https://install.pivpn.io | bash
# Choose WireGuard, port 51820/UDP, DNS of your choice, enable unattended upgrades
# Allow PiVPN to manage NAT and routing automatically when prompted
sudo systemctl status wg-quick@wg0
pivpn -d
```

### Set up DuckDNS Dynamic DNS
Create an account on [duckdns.org](https://www.duckdns.org). Then:
```bash
mkdir -p ~/duckdns && cd ~/duckdns
nano duck.sh
```
Paste and update with your domain/token:
```bash
echo url="https://www.duckdns.org/update?domains=YOURDOMAIN&token=YOURTOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -
```
Make executable:
```bash
chmod 700 duck.sh
```
Open cron:
```bash
crontab -e
```
Add:
```
*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1
```
Verify:
```bash
cat ~/duckdns/duck.log
```
Should return `ok`.  
After 5 minutes, confirm your IP at [DuckDNS Domains](https://www.duckdns.org/domains).

### Router Port Forwarding
Forward UDP 51820 → 192.168.0.10:51820
[![Router Port Forwarding](images/port-forwarding.png)](images/port-forwarding.png)

### Security Tools: UFW Firewall
```bash
sudo apt install ufw -y
sudo ufw allow 22/tcp
sudo ufw allow 51820/udp
sudo ufw enable
sudo ufw status
```

### Create VPN Client Profile(s)
```bash
pivpn add
pivpn -qr
```

### Test and Verify VPN Connection
1. Disconnect from your home Wi-Fi.  
2. Connect to an external network (LTE or public Wi-Fi).  
3. Activate the VPN connection in the WireGuard app.  
4. Verify the tunnel:
   - On the Raspberry Pi, check active peers:
     ```bash
     sudo wg show
     ```
   - On the client device, visit [whatismyipaddress.com](https://www.whatismyipaddress.com) — it should now display your 'home network’s public IP'.

## Troubleshooting
### SD Card Write Error
Use Raspberry Pi Imager → *Erase* option, then re-flash the image.

### SSH Connection Issues
If direct IP fails, use the hostname instead:
```bash
ssh username@<hostname>.local
```

### VPN Connected but No Internet
Check UFW settings:
```bash
sudo ufw status
sudo nano /etc/default/ufw
```
Set:
```
DEFAULT_FORWARD_POLICY="ACCEPT"
```
Then reload:
```bash
sudo ufw disable && sudo ufw enable
sudo systemctl restart wg-quick@wg0
```

## Resources
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- [PiVPN Installer](https://pivpn.io)
- [WireGuard Docs](https://www.wireguard.com/)
- [DuckDNS](https://www.duckdns.org/)

---
**Last Updated:** October 2025