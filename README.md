# homeserver
Home server setup

## Specs
* HPE ProLiant EC200a
* Xeon D1518
* 2x SK hynix 32GB DDR4 2666Mhz 
* 1x Samsung 980 nVME (rootfs)
* 2x Western Digital 6TB WD Red Plus CMR

## Installation
* Download and burn arch iso to usb
* vim /etc/pacman.conf :
```
ParallelDownloads = 25
```
* run ``archinstall``:
```
language: Engilsh
keyboard: us
mirror region: United States
harddrives: nvme1n1
bootloader: systemd-bootctl
hostname: bl-svr
root password: none
superuser account: bl
specify profile: minimum
select audio: none
select kernel: linux-lts
additional packages to install: 
network: copy settings from iso
select timezone: America/New_York
automatic time sync: yes
```

Chroot: yes
```
systemctl enable sshd
systemctl start sshd
```
Reboot

## Post Install
* SSH to the host as user
* vim /etc/pacman.conf :
```
Color
ParallelDownloads = 25
```
* ``sudo pacman -Syu`` just to make sure up to date

### Setup raid1 storage
* Install and start mdadm, create raid1 array
```
sudo pacman -S mdadm
sudo systemctl start mdadm
sudo systemctl enable mdadm``
sudo mdadm --create --verbose /dev/md0 --level=mirror --raid-devices=2 /dev/sda1 /dev/sdb1
```
* Create file partition
```
sudo fdisk /dev/md0
n <enter>
<enter>
<enter>
{exits fdisk}
sudo mkfs.ext4 /dev/md0
```
* Mount the newly created array, update fstab
```
sudo mkdir /mnt/md0
sudo mount /dev/md0 /mnt/md0
sudo blkid | grep /dev/md0
{copy that uuid}
sudo vim /etc/fstab
UUID={paste}}       /mnt/md0               ext4           defaults,noatime        0       1
{save}
```

# WATCH THE RAID ARRAY REBUILD
``mdadm --detail /dev/md0``
# WAIT FOR IT TO COMPLETE BEFORE CONTINUE

# Reboot

## Validate storage
* Login via ssh
* Check to make sure the storage mounted - should see something like this
```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/md2        126G   22G   98G  19% /
/dev/md3         28T   15T   11T  58% /home
/dev/md1        989M   86M  853M  10% /boot
```

# Install services
* Install the essentials:
```
sudo pacman -S vim git curl htop ansible fail2ban unzip docker docker-compose ffmpeg jq python-pip python-setuptools nload neofetch wireguard-tools rsync rclone

sudo systmctl start docker
sudo systemctl enable docker
sudo usermod -aG docker bl
```

## Build folder structure
```
mkdir /mnt/md0/backup
mkdir /mnt/md0/docker/
mkdir /mnt/md0/dev/
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

## Reboot again
* Login via ssh
* Check to make sure docker is running and permissions work: 
```
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

## Generate SSH keys
* ``ssh-keygen -t ed25519 -C bl-svr``
* ``cat ~/.ssh/id_ed25519.pub`` >> Github keys
* ``wget https://raw.githubusercontent.com/blurer/myBS/main/authorized_keys -P ~/.ssh/``
* ``mkdir ~/dev/ ; cd ~/dev ; git clone git@github.com:blurer/Homelab-Setup.git``
* ``cd ~/dev/Homelab-Setup ; ./main.py``

## Services to be installed:
* Plex
* ExpressVPN
* Torrent (connected to ExpressVPN)
* Portainer
* Nginx Proxy Manager
* Uptime Kuma
* Smokeping
* Mealie 
* Grocy
* Budget
* Speedtest Tracker
* OpenSpeedTest
* IPerf

