# Old Laptop/Computer Homeserver Guide:

## Specs and Model of the Old Laptop I'm using:
Aspire E1-571G

Intel Core(R) Core i5-3230M (4) @3.20 Ghz | Intel 3rd Gen Core processor Graphics Controller @1.10GHz (integrated).

NVIDIA GeForce 610M/710M/810M/820M / GT 620M 625M / 630M / 720M (Discrete).

4GB DDR3 and 500GB HDD.


## Software specifications:
Debian 13.6.0 amd64 netinst and docker containers to run each service in.


## Main Goals
#### PS: I Might add or remove goals along the process. However until I have a functional homeserver, I will live update this repo and readme. 
#### Some of these might require an addition ssd/nvme as well as more RAM, so I will most probably upgrade the machine along the way.
### PiHole:
A network-wide ad blocker and privacy tool that acts as a Domain Name System (DNS) sinkhole for all devices connected to your home network.
### JellyFin:
A volunteer-built media solution that allows you to stream to any device from the *totally legal license bought* media collection you have on your server. 
### WebRevival website:
Retro web page or community platform reminiscent of the early 1990s and early 2000s days of the internet before corporate takeover.
### Miniflux:
a minimalist, open-source, and self-hosted RSS and Atom feed reader designed for speed, simplicity, and distraction-free reading.
### Lemmy
A free, open-source, and decentralized alternative to Reddit and Hacker News for hosting discussion forums and sharing links.
### Small Luanti Server
A free, open-source voxel game engine and creation platform. Paired with a game like MineClonia, it provides extremely similar Minecraft-like gameplay and mechanics while being free, open source, and lightweight.
### PairDrop:
A local, web-based file-sharing tool inspired by Apple’s AirDrop that lets you transfer files instantly between devices on the same network.


## OS Install:
After flashing the debian iso from debian.org on the USB flash/thumb drive and booting into the graphical install, 
### 1- Language
Select your language of choice
### 2- Network: 
Choose your primary network interface for the install. For a homeserver, wired Ethernet is highly recommended. However if you aren't close to one, you can use wireless internet temporarily for the install.
### 3- Host&Domain Name:
For hostname, I'm going to name it "homeserver", but you can choose whatever you want.
For domainname, leave it blank or just enter "local" or "home".
### 4- Login Credentials:
Enter root password. (You will need this to run commands that require administrative (sudo) privileges.
Enter full name, username, and add a password
### 5- Partitioning:
#### Use Entire Disk & LVM:
There are multiple options for partitioning the disk. I ended up choosing "Guided - use entire disk and set up LVM" because it wipes the drive and sets up a flexible, virtual storage pool. It also allows partitions to be resized dynamically or expanded across new physical drives later.
#### Writing changes to disk
Make sure to select the hard drive for partitioning not the thumb drive and then select "All files in one partition (recommended for new users)", and finally write the current partitioning scheme to disk, then select yes twice to finish write changes to disk.
#### If Error: "No root file system is define. Please correct this from the partitioning menu"
then you might has accidentally press enter again after selecting yes to write changes to disk the first time causing you to select no the second time. Go back and redo. 
Select enter to the amount of volume group to use for guided partitioning. Now select "Finish Partitioning" and finally, select yes to write changes to disks. 
#### Writing changes to disk (fr):
At this point, it should start writing to the disk. Wait like 5 minutes and you will be taken to the next installation steps:
### 6- Configuring the package manager. 
The goal is to find a mirror of the Debian archive that is close to you on the network - be aware that nearby countries, or even your own, may not be the best choice. In my case, I choose to go with the German mirror ftp.de.debian.org. Wait for debian to install core system.
### 7- Software Selection:
To keep this as lightweight and minimal as possible, we will keep standard system utils ticked, tick "SSH server", and untick "Debian desktop environment" and any desktop environment, which leaves us with just the standard system utilities and SSH server ticked. 
### 8- Grub:
Since we wiped the drive during paritioning, it should be safe to install grub. Select yes to do so and wait for the installation to finish.
### 9- Boot:
After installation, your system should reboot. After, simply enter os through the grub selection menu by simply pressing enter and you should be greeted by cli. Congrats, you just installed Debian. Simply enter username and password and you are in.

## Setup
From this step onwards, you can get remove root access into the freshly installed Debian system via your main computer with ssh 
### (Optional) Disable dgpu:
Before booting into the OS, go to the bios and disable the GPU there. This only applies to laptops like this old acer where it's dgpu won't help with anything and will just draw exra heat and electricity.
### Login to Root User
```bash
su -
```
*(Enter your root password when prompted)*

### Update Apt and Install Sudo
```bash
apt update && apt install sudo -y
```

### Add Yourself to the Sudoers File
```bash
usermod -aG sudo username
```
*(Replace "username" with your actual Debian account username)*

### Exit and Apply Changes
```bash
exit
newgrp sudo
```

### Disable the Laptop Lid Sleep
Open the login configuration file:
```bash
sudo nano /etc/systemd/logind.conf
```
Find the line `#HandleLidSwitch=suspend`, remove the `#` symbol, and change it to:
```text
HandleLidSwitch=ignore
```
Press `Ctrl+O` then `Enter` to save, and `Ctrl+X` to exit the editor.

Restart the systemd logging service to apply changes instantly:
```bash
sudo systemctl restart systemd-logind
```

---

## Install Docker
```bash
sudo apt-get update && sudo apt-get install -y ca-certificates curl
curl -fsSL https://docker.com -o get-docker.sh
sudo sh get-docker.sh
rm get-docker.sh
```

---

## Free Network Port 53
Debian includes a default DNS service called `systemd-resolved` that claims network Port 53. You must disable this native daemon before launching Pi-hole, or the container will fail to start due to a port conflict.

### Stop and Disable System DNS Daemons
```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

---

## Pihole:

## Directory Structure
To keep our data organized and ready for potential future SSD storage upgrades, create a dedicated directory structure for the container services:

### Pihole Directories
```bash
mkdir -p ~/homeserver/pihole/
cd ~/homeserver/pihole
```

### Launching the Container
Run this single-line command to deploy Pi-hole immediately without needing to format a YAML configuration file:

```bash
sudo docker run -d \
  --name pihole \
  -p 53:53/tcp \
  -p 53:53/udp \
  -p 80:80/tcp \
  -e TZ='Europe/London' \
  -e FTLCONF_webserver_api_password='your_secure_password' \
  -e FTLCONF_dns_listeningMode='ALL' \
  -v /root/homeserver/pihole/config:/etc/pihole \
  --restart=unless-stopped \
  pihole/pihole:latest
```

## Post-Installation Verification
1. **Check Service Status:** Verify the container is running.
   ```bash
   sudo docker ps
   ```
2. **Access the Dashboard:** Open a browser and go to `http://<YOUR_SERVER_IP>/admin`.
3. **Connect Clients:** You could configure the router to go through the pihole before devices however for now I'm just going to manually change my devices' IPV4 into the IP address of our little server.

## Jellyfin:
    
### Jellyfin Directories:
    mkdir -p ~/homeserver/jellyfin/config
    mkdir -p ~/homeserver/jellyfin/cache
    mkdir -p ~/homeserver/jellyfin/media/movies
    mkdir -p ~/homeserver/jellyfin/media/tvshows

### Launching the Container:
    sudo docker run -d \
    --name jellyfin \
    --net=host \
    -v /root/homeserver/jellyfin/config:/config \
    -v /root/homeserver/jellyfin/cache:/cache \
    -v /root/homeserver/jellyfin/media:/media:ro \
    --restart=unless-stopped \
    jellyfin/jellyfin:latest


## Jellyfin Setup:

Go to a web browser on your main computer and paste

    http://<SERVERIP>:8096
    
*replace "SERVERIP" with you14 Year Old Nerd From Lebanon. I Love Programming, GNU/Linux and Other Open Source Operating Systems, As Well As Repurposing Old Tech.r server's actual local ip* 


Now that you have opened Jellyfin in your browser, you will see the Jellyfin Welcome Screen. Simply set a root username and password, add your media files, choose preferred language, and make sure to allow remote connections and enable automatic port mapping (UPnP).


To connect your additional devices to the server and access all your media, simply download the official Jellyfin software available android, ios, windows, linux, and macos. 


For IOS, AppleTV, and Apple Sillicon Macs the third party app Swiftfin is highly recommended for a faster native experience.
