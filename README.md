# pi-home-server

### 🛡️ Basic setup and security

```bash
# update system
sudo apt update && sudo apt upgrade -y

# install and configure ufw (firewall)
sudo apt install ufw -y
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

---

### 🐳 Docker & docker compose

```bash
# install docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER  # login again or reboot

# install docker compose
sudo apt install docker-compose -y
```

---

### 📦 Portainer (docker UI)

```bash
docker volume create portainer_data
docker run -d -p 9000:9000 -p 9443:9443 \
    --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:latest
```

---

### 🌐 WireGuard (VPN)

```bash
sudo apt install wireguard -y
```

> follow installation instructions in dialog. Dont forget port forwarting in your router (should basically be UDP 51820).

---

### Setup docker container

1. Create a folder and inside create a `docker-compose.yml` with the configuration from the `yml` inside this repository.
2. Then cd into the directory and run `docker compose up -d`

---

### 🔐 Advanced security (fail2ban, ssh config, updates)

```bash
# fail2ban against ssh-attacks
sudo apt install fail2ban -y

# enable automatic updates
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

---

### ✅ tips

* reboot: `sudo reboot`
* access to:

  * Portainer: `http://<raspberry-ip>:9000`
  * Homarr: `http://<raspberry-ip>:7575`

---

## More Packages

### [Cockpit](https://cockpit-project.org/running.html)

```bash
# install and auto-start cockpit
sudo apt update -y
sudo apt install cockpit -y
sudo ufw allow 9090/tcp
sudo systemctl enable --now cockpit.socket
# or
sudo systemctl start cockpit
sudo systemctl enable cockpit
```


## Documentation and credits
- https://techhut.tv/must-have-home-server-services-2025/
- https://github.com/TechHutTV/homelab