# How to setup Immich on a Raspberry PI5 using Dock, Portainer and Nginx Proxy Manager (Let's encrypt Certificates)
<br/>
<p></p>

## Initial information

First I would like to warn you about that fact that my Raspberry PI5 was massively unstable in this setup. I tried multiple configurations all with the official power supply 5v 5a.
After much pain and suffering I decided to go for a different hardware solution. I went for a slightly bigger box, HP Prodesk 600 SFF G5, i5-9500, 16gb ram, a bit noisy when doing AI workloads.<br/>
<br/>
You can use the same guide for Debian installation<br/>
<br/>
<br/>
If you still want to try setting up Raspberry PI5, I used the following:
- Raspberry Pi 5
- Official Raspberry PI 5 power adapter (or the one with the PI Hat)
- [PI Hat for M.2 SSD](https://www.aliexpress.com/item/1005006383425420.html?spm=a2g0o.order_detail.order_detail_item.3.25086d76ShtfSJ)
- [PI Case designed for PI Hat](https://www.aliexpress.com/item/1005006383425420.html?spm=a2g0o.order_detail.order_detail_item.5.25086d76ShtfSJ)
- M.2 disk
- USB External HDD 3.5" 4TB (for storing pictures and videos)
- [M.2 to USB device](https://www.aliexpress.com/item/1005001524803408.html?spm=a2g0o.order_list.order_list_main.51.54721802hr2nsZ)
- Micro SD card

<br/>
<br/>
<p></p>

## INITIAL SETUP of the PI5 (20 minutes) (External guide)

Assemble your PI5 withut the case (you probably need a few tries, so don't use the case in the beginning)
Plug your Micro SD card in a PC/MAC, download the Raspberry PI5 image and create an installation media using "Raspberry PI5 imager".
Insert the SD card, do not insert the M.2 disk.

YOUR_USERNAME is not in the sudoers file.
```
su -
usermod -aG sudo YOUR_USERNAME
exit
```

You can start with updating the system
```
sudo apt update && sudo apt upgrade -y
```
Update the firmware of your PI5
```
sudo rpi-eeprom-update -a
```
Reboot yor PI5
```
sudo reboot
```

Go here for rest of the guide:
https://wiki.geekworm.com/NVMe_SSD_boot_with_the_Raspberry_Pi_5
<br/>
<br/>
<p></p>

## SETUP External USB disk (10 minutes):

Connect to your PI5 with Putty or something else.
Then run these commands in the terminal.

List out disk info
```
lsblk
```

It might look something like this, sdb is my 4TB external disk
<pre>
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    1 28.6G  0 disk
├─sda1   8:1    1  512M  0 part /boot/firmware
└─sda2   8:2    1 28.1G  0 part /
sdb      8:16   0  3.6T  0 disk
</pre>

Create the partition:
```
sudo fdisk /dev/sdb
```

You will get a warning if it's a disk larger than 2GB, no worries.
<pre>
Within FDISK, press:
    g          ...to create GPT disklabel
    n          ...to create a new partition
    (enter)    ...when asked for Partition number
    (enter)    ...when asked for First sector
    (enter)    ...when asked for Last sector
    w          ...to write/save changes
</pre>

List out disk info, again
```
lsblk
```
It should look more like this now
<pre>
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    1 28.6G  0 disk
├─sda1   8:1    1  512M  0 part /boot/firmware
└─sda2   8:2    1 28.1G  0 part /
sdb      8:16   0  3.6T  0 disk
└─sdb1   8:17   0  3.6T  0 part
</pre>

See more disk info:
```
lsblk -f
```
NOTE that fstype on sdb1 is missing
<pre>
NAME      FSTYPE FSVER LABEL     UUID                                     FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1    vfat   FAT32 bootfs    50C8-AEAE                                436.4M   14% /boot/firmware
└─sda2    ext4   1.0   rootfs    fc7a1f9e-4967-4f41-a1f5-1b5927e6c5f9     19.9G    23% /
sdb
└─sdb1
</pre>

Format the disk and label it / give it a name: (replace usb4TB with your diskname)
```
sudo mkfs.ext4 -L usb4TB /dev/sdb1
```

See more diskinfo, again:
```
lsblk -f
```
NOTE that fstype on sdb1 is now present and has proper label!
<pre>
NAME      FSTYPE FSVER LABEL    UUID                                     FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1    vfat   FAT32 bootfs    50C8-AEAE                               436.4M   14% /boot/firmware
└─sda2    ext4   1.0   rootfs    fc7a1f9e-4967-4f41-a1f5-1b5927e6c5f9    19.9G    23% /
sdb
└─sdb1    ext4   1.0   usb4TB    ef63304c-d37c-495b-b381-4bb5d20110ce
</pre>

Creating Mount point:
NOTE: Replace usb4TB with your label
```
sudo mkdir /media/usb4TB
```
Change owner of usb4TB folder
```
sudo chown [username]:[username] /media/usb4TB
```

Go to /media folder and type
```
ls -al
```
to verify that your user has ownership of the folder (all other folders here should be owned by root)

Mounting the usb drive
```
sudo mount /dev/sdb1 /media/usb4TB
```

Verify that it's mounted correctly
```
df -h
```
It sould look something like this, note bottom line information
<pre>
Filesystem      Size  Used Avail Use% Mounted on
udev            3.8G     0  3.8G   0% /dev
tmpfs           805M  6.7M  799M   1% /run
/dev/sda2        28G  6.3G   20G  24% /
tmpfs           4.0G  368K  4.0G   1% /dev/shm
tmpfs           5.0M   48K  5.0M   1% /run/lock
/dev/sda1       510M   74M  437M  15% /boot/firmware
tmpfs           805M  144K  805M   1% /run/user/1000
/dev/sdb1       3.6T   28K  3.4T   1% /media/usb4TB
</pre>

Add Automounting of the drive on boot
```
sudo nano /etc/fstab
```
Add the following line at the bottom of the file
```
/dev/sdb1   /media/usb4TB     ext4    defaults    0    0
```
Reboot to verify that everything works
```
sudo reboot
```

<br/>
<br/>
<p></p>

## OPTIONAL INSTALL Samba (5 minutes):
Install Samba
```
sudo apt-get install samba samba-common-bin
```
OPTIONAL<br/>
If you need to remove old directory (replace tor and usb4TB with your names)
```
sudo rm -R "/media/tor/usb4TB/immich"
```
Creating directory (don't use SUDO here) (replace tor and usb4TB with your names)
```
mkdir "/media/tor/usb4TB/immich"
mkdir "/media/tor/usb4TB/immich/external"
```
Creating the share configuration-file
```
sudo nano /etc/samba/smb.conf
```
Paste information in the conf file (replace tor and usb4TB with your names)
```
[immich]
path = /media/tor/usb4TB/immich
writeable=Yes
create mask=0777
directory mask=0777
public=no
```
Create a samba share user
```
sudo smbpasswd -a yourusername
```
When prompted, set the desired password

Restart Samba service:
```
sudo systemctl restart smbd
```
OPTIONAL<br/>
If you need to know host IP address
```
hostname -I
```
ADDITIONAL INFO<br/>
Since you have a share at \\192.168.1.111\immich, you can easily set up a backup job if you have a server on your network. This backup will also include the \immich\external folder, so it should be a complete backup of every photo and video.

<br/>
<br/>
<p></p>

## OPTIONAL MAP Network share for push backup
Install CIFS, smb client
```
sudo apt install cifs-utils
```
Create mounting folder
```
sudo mkdir /media/share
```
Create credentials file
```
nano /root/.smbcredentials
```
Fill in this in the credentials file
```
username=smb_username
password=smb_password
```
Restrict access to the credentials file
```
sudo chmod 400 /root/.smbcredentials
```
Mount the share
INFO: Here it's specified version 3.0, if you have an older system 1.0 might be used
```
sudo mount -t cifs -o rw,vers=3.0,credentials=/root/.smbcredentials //192.168.1.111/share /media/share
```
Set up AutoMount on reboot of the share
```
sudo nano /etc/fstab
```
Add this line at the end of your file (v1.0 AND v3.0 example)
```
//192.168.1.110/Share /media/share cifs vers=3.0,credentials=/root/.smbcredentials,_netdev,x-systemd.automount 0 0
//192.168.1.111/USB3/sda2 /media/share2 cifs vers=1.0,credentials=/root/.smbcredentials,_netdev,noauto,x-systemd.automount 0 0
```
Create Scheduled task for backup every night
```
crontab -e
```
Paste the following to run the rsync job every night at 01:00
```
0 1 * * *  rsync -vr /media/usb4TB/immich/ /media/share/immich
```

<br/>
<br/>
<p></p>

## INSTALL Docker (3 minutes):

```
sudo apt update && sudo apt upgrade -y
sudo apt install curl -y
curl -sSL https://get.docker.com | sh
```
Add user to docker group (replace yourusername)
```
sudo adduser yourusername docker
newgrp docker
```
newgrp is for refreshing so you don't have to log out and in again.
<br/>
<br/>
<p></p>

## PREPARING your OS for larger docker images (required for immich on Debian OS)

Check what your var directory is set up with
```
df -h
```
<pre>
Filesystem                  Size  Used Avail Use% Mounted on
udev                        7.7G     0  7.7G   0% /dev
tmpfs                       1.6G  1.8M  1.6G   1% /run
/dev/mapper/vg_system-root   16G   13G  1.4G  91% /
tmpfs                       7.7G     0  7.7G   0% /dev/shm
tmpfs                       5.0M  8.0K  5.0M   1% /run/lock
/dev/nvme0n1p2              456M  145M  287M  34% /boot
/dev/mapper/vg_system-home   30G   35M   29G   1% /home
/dev/mapper/vg_system-var   5.1G  2.9G  2.0G  60% /var
/dev/nvme0n1p1              511M  5.9M  506M   2% /boot/efi
tmpfs                       1.6G  112K  1.6G   1% /run/user/1000
</pre>
Mine is /dev/mapper/vg_system-var

Give it some more space:
INFO1 : Replace /dev/mapper/vg_system-var with other path if your system is set up with another path
INFO2 : I extended the disk with 25GB additional storage
```
sudo lvextend -L +25G /dev/mapper/vg_system-var
sudo resize2fs /dev/mapper/vg_system-var
```

Check to see that the space was given ok
```
df -h
```
<pre>
Filesystem                  Size  Used Avail Use% Mounted on
udev                        7.7G     0  7.7G   0% /dev
tmpfs                       1.6G  1.8M  1.6G   1% /run
/dev/mapper/vg_system-root   16G   13G  1.4G  91% /
tmpfs                       7.7G     0  7.7G   0% /dev/shm
tmpfs                       5.0M  8.0K  5.0M   1% /run/lock
/dev/nvme0n1p2              456M  145M  287M  34% /boot
/dev/mapper/vg_system-home   30G   35M   29G   1% /home
/dev/mapper/vg_system-var    30G  2.9G   26G  10% /var
/dev/nvme0n1p1              511M  5.9M  506M   2% /boot/efi
tmpfs                       1.6G  112K  1.6G   1% /run/user/1000
</pre>

## INSTALL Portainer (2 minutes):
Install Portainer (if you need SUDO here, then your user is missing from docker group.)
```
docker volume create portainer_data
```
```
docker run -d -p 8000:8000 -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```
OPTIONAL<br/>
If you like to store some data outside containers do this step, now you can store it in /srv
```
sudo chown -R myusername:myusername /srv
```
<br/>

ADDITIONAL<br/>
Updating Portainer to newest version
```
sudo apt update && sudo apt upgrade -y
```
```
docker stop portainer
```
```
docker rm portainer
```
```
docker pull portainer/portainer-ce:latest
```
```
docker run -d -p 8000:8000 -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```
<br/>
<br/>
<p></p>

## SETUP your external domain, this guide is for no-ip.org since it's free (5 minutes)
Go to www.no-ip.org
- Create a new user, or login to your existing one
- Click "Dynamic DNS"
- Click "NO-IP Hostnames"
- Click "Create Hostname"
- Fill in Hostname "ExamplePrefix"
- Choose drop down menu "no-ip.org"
- Make sure "DNS Host (A)" is selected
- Fill in your EXTERNAL IP (no-ip will display it in the textbox as background, so just read it, then type it back in)

## SETUP your router port forwarding (5 minutes)
This guide is only for the router i use, but it should be a similar process for you.
Go to 192.168.1.1 (this is usually the ip for your router)
- I Login
- I Click "Advanced"
- I Click "Advanced Setup"
- I Click "Port Forwarding / Port Triggering"
- I Make sure I have "Port Forwarding" selected

HTTPS

- I Click "Add Custom Service"
- I Fill inn Service Name "Immich-HTTPS"
- I Make sure I have "TCP/UDP" selected
- I Fill inn External Port Range "443"
- I Fill inn Internal Port Range "443"
- I Fill inn the IP Address for my Raspberry PI5 "192.168.1.111"

OPTIONAL<br/>
HTTP

- I Click "Add Custom Service"
- I Fill inn Service Name "Immich-HTTP"
- I Make sure I have "TCP/UDP" selected
- I Fill inn External Port Range "80"
- I Fill inn Internal Port Range "80"
- I Fill inn the IP Address for my Raspberry PI5 "192.168.1.111"
<br/>
<br/>
<p></p>

## INSTALL nginex-proxy-manager on Portainer (10 minutes)
Go to Portainer web: 192.168.1.111:9000
(IP here, and rest of doc, is only as an example, you can find your ip using: hostname -I)
- Create a new user and login
- Click "Home"
- Click "Live Connect" (on your right side)
- Click "Networks"
- Click "Add Network"
- Fill in name "npm-network"
- Click "Create the Network"

Next

- Click Stacks
- Click "Add Stack"
- Fill in name "nginx-proxy-manager"
- Make sure "Web Editor" is selected
- Fill in CODE BELOW:
```
version: '3.8'
services:
  nginx-proxy-manager:
    container_name: nginx-proxy-manager
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - /srv/nginx-proxy-manager:/data
      - /srv/letsencrypt:/etc/letsencrypt
```
- Click "Deploy the stack"

ADDITIONAL INFO<br/>
Since we use /srv in volumes, this means that if you, at a later time, remove the stack and the volumes, the data will persist since its stored outside docker at location /srv

<hr>

### Go to Nginx Proxy Manager web: 192.168.1.111:81

Default username:
```
admin@example.com
```
Default password:
```
changeme
```
- Click "Hosts"
- Click "Proxy Hosts"
- Click "Add proxy host"
- Fill in Domain Names EXAMPLE "myfreedomainname.no-ip.org"
- Fill in Forward Hostname/IP EXAMPLE "192.168.1.111"
- Fill in Forward Port "2283"
- Enable "Websockets Support"
- Click "Save"
- Click "..."
- Click "Edit"
- Click "SSL"
- Change "None" to "Request Certificate"
- Enable "Force SSL"
- Enable "HTTP/2 Support"
- Click "Save"

ADDITIONAL INFO<br/>
Let's encrypt can issue 5 SSL certificates within the same couple of days, so if you are testing and need to reset your whole box and re-issue for certificates, you have 5 tries for that domain.
<br/>
<br/>
<p></p>

## INSTALL immich on PORTAINER (10 minutes):
ADDITIONAL INFO<br/>
Library in immich is readonly, you can not change anything. I would recommend using CLI to import your existing photos and videos instead of using library.

Go to Portainer web: 192.168.1.111:9000
- Click Stacks
- Click "Add Stack"
- Fill in name "immich"
- Make sure "Web Editor" is selected
- Fill in CODE BELOW:

```
name: immich
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    command: ['start.sh', 'immich']
    volumes:
      - ${UPLOAD_LOCATION}:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - stack.env
    ports:
      - 2283:3001
    depends_on:
      - redis
      - database
    restart: always
  immich-microservices:
    container_name: immich_microservices
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    command: ['start.sh', 'microservices']
    volumes:
      - ${UPLOAD_LOCATION}:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - stack.env
    depends_on:
      - redis
      - database
    restart: always
  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    volumes:
      - model-cache:/cache
    env_file:
      - stack.env
    restart: always
  redis:
    container_name: immich_redis
    image: registry.hub.docker.com/library/redis:6.2-alpine@sha256:51d6c56749a4243096327e3fb964a48ed92254357108449cb6e23999c37773c5
    restart: always
  database:
    container_name: immich_postgres
    image: registry.hub.docker.com/tensorchord/pgvecto-rs:pg14-v0.2.0@sha256:90724186f0a3517cf6914295b5ab410db9ce23190a2d9d0b9dd6463e3fa298f0
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
    volumes:
      - /srv/immich_pgdata:/var/lib/postgresql/data
    restart: always
volumes:
  pgdata:
  model-cache:
```
- Click "Advanced mode"
- Fill in CODE BELOW: (replace tor and usb4TB with your names, also set a new password)
```
UPLOAD_LOCATION=/media/tor/usb4TB/immich
DB_PASSWORD=changeme
IMMICH_VERSION=release
DB_HOSTNAME=immich_postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
REDIS_HOSTNAME=immich_redis
```
- Click "Simple mode" (to switch back, optional)
- Click "Deploy the stack"

Go to Immich web: 192.168.1.111:2283
- Set up new user
- Login

OPTIONAL<br/>
Setup External Library in the GUI, you need both docker-compose.yml (pasted code to Web Editor) and the GUI for it to show up. External Library is for viewing ONLY, you cannot delete files (even in the GUI). I like immich-cli command for importing better, this gives full control of the imported files.
- Click "Administration"
- Click "External Library"
- Click "Create Library"
- Click "Create"
- Click "..."
- Click "Edit Import Paths"
- Fill in "/media/tor/usb4TB/immich/external"
- Click "Save"


OPTIONAL<br/>
Installing immich-cli in Powershell:
- Install [Node.JS bundled with NPM](https://nodejs.org/en/download), download and run the setup.
- Install the new @immich-cli in powershell
```
npm i -g @immich/cli
```

Find your users API key

Go to Immich web: 192.168.1.111:2283
- Click "Your User Icon" (Top right corner)
- Click "Account Settings"
- Click "API Keys"
- Click "Create Key"
- Copy the key
- Click "Done"

Login to your immich server
```
immich login https://192.168.1.111/api YourApiKeyHere
```
Upload files to your immich server
```
immich upload --recursive 'F:\ExampleDir\ExampleUser1'
```
<br/>
<br/>
<p></p>

## ADDITIONAL Backup all the stuff
Manual backup<br/>
Backup all the immich image and video files
```
rsync -av /media/usb4TB/immich/library/ /media/share/immich/library
```
Backup the immich database
```
sudo rsync -av /srv/immich_pgdata/ /media/share/immich/immich_pgdata
```
Backup nginx certificates
```
sudo rsync -av /srv/letsencrypt/ /media/share/nginx-proxy-manager/letsencrypt
```
Backup nginx data
```
sudo rsync -av /srv/nginx-proxy-manager/ /media/share/nginx-proxy-manager/data
```
<br/>

RSync as cron sheduled job to backup all the immich image and video files and dababase
```
sudo nano /etc/crontab
```
Paste this at bottom of all the other text, DO NOT DELETE ANYTHING HERE
Change yourusername
```
#
#Backuplocation 1 - SMB 3.0
0   1    * * *   root    rsync -av /srv/letsencrypt/ /media/share/nginx-proxy-manager/letsencrypt
0   1    * * *   root    rsync -av /srv/nginx-proxy-manager/ /media/share/nginx-proxy-manager/data
0   1    * * *   root    rsync -av /srv/immich_pgdata/ /media/share/immich/immich_pgdata
10  1    * * *   yourusername rsync -av --progress /media/usb4TB/immich/library/ /media/share/immich/library
#
#Backuplocation 2 - SMB 1.0
30   1    * * *   root    rsync -rv /srv/letsencrypt/ /media/share2/nginx-proxy-manager/letsencrypt
30   1    * * *   root    rsync -rv /srv/nginx-proxy-manager/ /media/share2/nginx-proxy-manager/data
30   1    * * *   root    rsync -rv /srv/immich_pgdata/ /media/share2/immich/immich_pgdata
40   1    * * *   yourusername rsync -rv --ignore-existing --progress /media/usb4TB/immich/library/ /media/share2/immich/library
```
<br/>
<br/>
<p></p>

## ADDITIONAL Restore immich from backup
RSync to restore images from your backup
```
rsync -av /media/share/immich/library/ /media/usb4TB/immich/library
```
RSync to restore database files from your backup
```
sudo rm -R /srv/immich_pgdata
mkdir immich_pgdata
sudo rsync -av /media/share/immich/immich_pgdata/ /srv/immich_pgdata
```
INFO<br/>
Since my example is on an external harddrive with smbv1.0, i need to take ownership of the restored folder structure
```
sudo chown -R yourusername:yourusername /srv/immich_pgdata
```
<br/>
<br/>
<p></p>

## ADDITIONAL Update all the stuff, manual
Immich Update procedyre using portainer
If you dont have the config saved somewere, first copy the text and paste it in notepad or somewhere else for easy copy back
- Click "Stacks"
- Click "immich"
- Click "Editor"
- Copy config text and variables as text in advanced view to notepad or something else
- Click "Stack"
- Click "Stop this stack"
- Click "Delete this stack"
- Click "Images"
- Select all images that have yellow label "Unused"
- Click "Remove"
- Click "Stacks"
- Click "Add stack"
- Fill inn like you do in normal immich install, now it will pull the newest version

## ADDITIONAL Update all the stuff, script
I have not tried this yet. I will try it at next update.
```
arrImmich=$( docker ps --filter "name=immich" -q )
docker stop $arrImmich
docker rmi $(docker images ghcr.io/immich-app/immich-server:release -q)
docker rmi $(docker images ghcr.io/immich-app/immich-machine-learning:release -q)
docker rmi $(docker images registry.hub.docker.com/tensorchord/pgvecto-rs -q)
docker rmi $(docker images registry.hub.docker.com/library/redis -q)
docker start $arrImmich
```

<br/>
<br/>
<p></p>

## Install Pre-req and immich-CLI
How to install and use immich-cli in powershell
- Download and install Node JS https://nodejs.org/en/download
- Install NPM in Powershell (As Administrator)
```
npm install -g npm@10.7.0
```
- Install immich-cli
```
npm i -g @immich/cli
```
- Find your API-key, log in to webpage, click profile picture top right corner, Account Settings, API Keys, new key, copy the new key
- Login using your API-key
```
immich login http://192.168.1.111:2283/api FDSrewgArtqrwefASDFewrdFASDfweqrRSDFAFewrEXAMPLE
```
- Upload using this command for best results
```
immich upload --include-hidden --recursive 'E:\Example Folder'
```

