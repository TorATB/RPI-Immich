# How to setup Immich on a Raspberry PI5 using Dock, Portainer and Nginx Proxy Manager (Let's encrypt Certificates)
<br/>
<p></p>

## Initial information

My hardware are as following:
- Raspberry Pi 5
- Official Raspberry PI 5 power adapter (or the one with the PI Hat)
Important with 5V AND 5A or else your system will be unstable!
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

Go here for rest of the guide:
https://wiki.geekworm.com/NVMe_SSD_boot_with_the_Raspberry_Pi_5
<br/>
<br/>
<p></p>

## SETUP External USB disk (10 minutes):

First I removed all the partitions in windows, as I'm a very beginner in Linux (PI OS), and set the drive as GPT.
Then Connect to your PI5 with Putty or something else.
Then run these commands in the terminal.

Create the partition:
```
sudo fdisk /dev/sda
```

<pre>
Within FDISK, press:
    d ...to delete the current partition (if you have any pratitions)
    n ...to create a new partition
    p ...to specify it as a PRIMARY partition
    1 ...to set it as the 1ST primary partition
    w ...to write the changes.
</pre>

Map the disk: (replace ex4TB with your diskname)
```
mkfs.ext4 -L ex4TB /dev/sda1
```

If you have any trouble accessing the disk with "Permission Denied"
Plug in the usb and go to /media/[yourusername] in the terminal.
Type ls -al and see if the USB mount point is owned by root.
If it is, type sudo chown [username]:[username] [mount point]
and enter your password when asked.
<br/>
<br/>
<p></p>

## INSTALL Samba (5 minutes):
Install Samba
```
sudo apt-get install samba samba-common-bin
```
OPTIONAL<br/>
If you need to remove old directory (replace tor and ex4TB with your names)
```
sudo rm -R "/media/tor/ex4TB/immich"
```
Creating directory (don't use SUDO here) (replace tor and ex4TB with your names)
```
mkdir "/media/tor/ex4TB/immich"
mkdir "/media/tor/ex4TB/immich/external"
```
Creating the share configuration-file
```
sudo nano /etc/samba/smb.conf
```
Paste information in the conf file (replace tor and ex4TB with your names)
```
[immich]
path = /media/tor/ex4TB/immich
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


## INSTALL Docker (3 minutes):

```
sudo apt update && sudo apt upgrade -y
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

## INSTALL Portainer (2 minutes):
Install Portainer (if you need SUDO here, then your user is missing from docker group.)
```
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```
OPTIONAL<br/>
If you like to store some data outside containers do this step, now you can store it in /srv
```
sudo chown -R myusername:myusername /srv
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
- Create a new user
- Log in
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
I have added /media/tor/ex4TB/immich/external:/media/tor/ex4TB/immich/external as volumes as I used it for a while, but soon found out that it's a lot better to add bulk pictures and video using CLI command API. I decided to leave the two volume entries in this setup as you might want to use it. I recommend using CLI. (replace tor and ex4TB with your names)

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
      - /media/tor/ex4TB/immich/external:/media/tor/ex4TB/immich/external
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
      - /media/tor/ex4TB/immich/external:/media/tor/ex4TB/immich/external
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
      - pgdata:/var/lib/postgresql/data
    restart: always
volumes:
  pgdata:
  model-cache:
```
- Click "Advanced mode"
- Fill in CODE BELOW: (replace tor and ex4TB with your names, also set a new password)
```
UPLOAD_LOCATION=/media/tor/ex4TB/immich
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
- Fill in "/media/tor/ex4TB/immich/external"
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



