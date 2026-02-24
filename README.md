# Build your own Budget Home Server using Proxmox and Deploy Actual Budget

<img src="media/video-thumb.png" height="200">
https://www.youtube.com/watch?v=pVv-fFPuudc

## Hardware Checklist

1. Old Laptop / budget mini-PC / PC in working condition.
2. Screwdriver set that is suitable for your hardware - Laptop / mini-PC
3. (If using a PC or mini-PC), Monitor and keyboard, only for initial setup
4. USB Flash drive for installing Proxmox from - 16 Gb or higher
5. 2x SSDs - SATA or NVMe - 128 GB or higher. Used ones from ebay at 80% health works fine. Stick to reputable brands such as Samsung Evo or Pro models (very reliable), WD blue or black
6. (If using laptop that has only 1 SSD connector) - USB SSD enclosure - SATA or NVMe

## Milestones

1. Install Proxmox (the Hypervisor) on your computer
2. Install Debian OS inside a Virtual Machine (VM)
3. Install Docker inside Debian & deploy portainer
4. Deploy Actual Budget server

![Architecture](media/architecture-diagram.png)

## Dowload Links

- Ventoy: https://www.ventoy.net/en/download.html
- Balena Etcher: https://etcher.balena.io/
- Proxmox: https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso
- Debian: https://www.debian.org/

## Installing Proxmox

<img src="media/proxmox-logo.svg" height="50">

**BIOS / UEFI Key**:

- look up the model of your computer to find it
- In my Lenovo it was holding down the 'Fn' key and mashing the 'F2' key.
- On some computers, you may hear a beep

**Choice of File system**:

- Ideally, choose ZFS in Mirror (can withstand one drive failing at a time)
- Btrfs is also a good choice when using Mirror mode.

**Network**:

- Make sure to take a screenshot or note down the IP address of Proxmox.
- Don't worry if you didn't, it will be shown on the terminal on the screen on the server.
- In the future, we will assign a static IP to the server.

## Accessing Proxmox through it's Web UI

- Access using this format

```html
https://<replace_with_ip_address>:8006
```

**Warnings/Errors to ignore**

- HTTPS warning about your connection not being private
- Proxmox Enterprise repository warning

**SSH Setup**

<img src="media/debian-logo.png" height="100">

- From the Debian VM's Console tab (aka the shell) type

```bash
# Get the IP address of your Debian OS
ip a
# look for inet with number in the format X.X.X.X, skip the one that shows 127.0.0.1
```

- From the PVE node's Shell tab type

```bash
ssh <your_debian_username>@<debian_ip_address>
# enter password at next prompt
```

## Installing Docker inside Debian

<img src="media/docker-logo.png" height="100">

**Pre-setup**:

```bash
# Become root for the inital setup
su -

apt update
apt upgrade

apt install sudo

sudo usermod -aG sudo <your_docker_username>

# Log out of root
exit

sudo apt install curl

# Log out of your debian user's shell to apply changes
exit

# Log int your debian user's shell again
ssh <your_debian_username>@<ip_address>
# enter password at next prompt
```

**Installing Docker**:

```bash
sudo curl -fsSL https://get.docker.com | sh

# Check docker version (optional)
docker --version
```

## Installing Portainer

<img src="https://avatars.githubusercontent.com/u/22225832" height="100">


```bash
docker volume create portainer_data

docker run -d -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/

docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

Open a new tab in your browser and navigate to https://<debian_ip_address>:9443


## Deploying Actual Budget Server

<img src="media/actualbudget-logo.png" height="100">

SSH back into Debian via Proxmox node's shell

```bash
sudo mkdir -p /docker/volumes/actual-server/data

cd /docker/volumes/actual-server/data

# Using the official documentation https://actualbudget.org/docs/config/https/
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout selfhost.key -out selfhost.crt

sudo nano config.json
```

paste this into the json file:

```json
{
  "https": {
    "key": "/data/selfhost.key",
    "cert": "/data/selfhost.crt"
  }
}
```
save and exit

**Actual Budget - Docker Compose**:

(To be used inside the stacks screen in Portainer)

```yaml
services:
  actual_server:
    image: actualbudget/actual-server:latest
    ports:                      
    - '5006:5006'
    volumes:
    - /docker/volumes/actual-server/data:/data
    restart: unless-stopped
```

# == THE END ==
