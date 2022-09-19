# homeserver
Home server setup

## Specs
* HPE ProLiant EC200a
* Xeon D1518
* 2x SK hynix 32GB DDR4 2666Mhz 
* 1x Samsung 980 nVME (rootfs)
* 2x Western Digital 10TB WD Red Plus CMR


## Installation
* Download and burn proxmox iso to usb
* Boot to usb disk, install proxmox.

# Info

| Host | CT | Name | OS | IP/cidr |
|---|---|---|---|---|
| VM1^ | n/a | VM1 | Proxmox | 192.168.6.0/22|
| VM1 | 101 | dns1 | Ubu 20.04 | 192.168.6.10/22|
| VM1 | 102 | docker1| Ubu 20.04 | 192.168.6.11/22|
| VM1 | 103 | plex | Ubu 20.04 | 192.168.6.12/22|
| VM1 | 104 | bastion | Ubu 20.04 | 192.168.6.13/22|
| VM2^ | n/a | VM2 | Proxmox | 192.168.5.0/22 | 

*^ Cluster member*

# VM1 Post Install
* SSH to host
* Fix repo bs: ``bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-install.sh)"`` (reboot after)
* Darkmode: ``bash <(curl -s https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh ) install``
* Setup zfs pool for 2x10TB HDDs
* Download Ubuntu 20.04 Server LXC
* Start building VMs

# DNS
```
ct: 100
hostname: dns1
vcpu: 1
mem: 512m
storage 1: 8g (nvme)
IP: 192.168.6.10/22
```
## Setup
```
apt update -y
apt upgrade -y
apt install vim git curl rsync htop nload -y
curl -sSL https://install.pi-hole.net | bash
```
*Reboot CT*
* Test DNS is working.
* Add whitelists: https://raw.githubusercontent.com/blurer/Homelab-Setup/main/pihole/whitelist.txt
* Add blacklists: https://raw.githubusercontent.com/blurer/Homelab-Setup/main/pihole/blocks.txt
* Update the lists: ``docker exec -it pihole /bin/bash`` -> ``pihole -g`` -> ``exit``
* Setup router to use ``192.168.6.10`` as DNS resolver
* Reboot

# Docker
```
ct: 101
hostname: docker1
mem: 16g
storage 1: 128g (nvme)
storage 2: 512g (hdd)
IP: 192.168.6.11/22
```
## Setup
```
apt update -y
apt upgrade -y
apt install vim git curl rsync htop nload -y
curl -fsSL https://get.docker.com -o get-docker.sh
bash get-docker.sh
systemctl enable docker
apt install docker-compose -y
```
*Reboot ct*


# Plex VM
```
ct: 102
hostname: plex
vcpu: 2
mem: 8g
storage 1: 32g (nvme)
storage 2: 8T (hdd)
IP: 192.168.6.12/22
```
## Setup
```
apt update -y
apt upgrade -y
apt install vim git curl rsync htop nload python3-pip python3-setuptools gnupg -y
pip3 install bpytop
curl https://downloads.plex.tv/plex-keys/PlexSign.key | sudo apt-key add -
echo deb https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
sudo apt update -y
sudo apt install plexmediaserver -y
sudo systemctl enable plexmediaserver
sudo systemctl start plexmediaserver
```
*Reboot*
```
mkdir /mnt/md0/media/
mkdir /mnt/md0/media/tv/
mkdir /mnt/md0/media/tv/2160p/
mkdir /mnt/md0/media/tv/1080p/
mkdir /mnt/md0/media/tv/720p/
mkdir /mnt/md0/media/movies/
mkdir /mnt/md0/media/movies/2160p/
mkdir /mnt/md0/media/movies/1080p/
mkdir /mnt/md0/media/movies/720p/
mkdir /mnt/md0/media/music/
mkdir /mnt/md0/media/family/
mkdir /mnt/md0/media/family/grandparents/
mkdir /mnt/md0/media/family/kids/
mkdir /mnt/md0/media/education/
mkdir /mnt/md0/media/bl/
mkdir /mnt/md0/media/audiobooks/
mkdir /mnt/md0/media/podcasts/
mkdir /mnt/md0/media/foreign/
mkdir /mnt/md0/media/foreign/tagalog/
mkdir /mnt/md0/media/foreign/tagalog/tv/
mkdir /mnt/md0/media/foreign/tagalog/movies/
mkdir /mnt/md0/media/foreign/japanese/
mkdir /mnt/md0/media/foreign/japanese/tv/
mkdir /mnt/md0/media/foreign/japanese/movies/
mkdir /mnt/md0/seed/
mkdir /mnt/md0/tmp/
```

