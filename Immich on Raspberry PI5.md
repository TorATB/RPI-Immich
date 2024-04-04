# How to setup Immich on a Raspberry PI5 using Dock, Prtainer and Nginx Proxy Manager for Certificates

## Initial information

My hardware are as following:
- Raspberry Pi 5
- Official Raspberry PI 5 power adapter (or the one with the PI Hat)
Important with 5V AND 5A or else your system will be unstable!
- PI Hat for M.2 SSD - https://www.aliexpress.com/item/1005006383425420.html?spm=a2g0o.order_detail.order_detail_item.3.25086d76ShtfSJ
- PI Case designed for PI Hat - https://www.aliexpress.com/item/1005006383425420.html?spm=a2g0o.order_detail.order_detail_item.5.25086d76ShtfSJ
- M.2 disk
- USB External HDD 3.5" 4TB (for storing pictures and videos)

## SETUP disk:

I removed all the partitions in windows, as I'm a very beginner in Linux (PI OS), and set the drive as GPT.

Create the partition:
```
sudo fdisk /dev/sda
```
Within FDISK, press:
    d ...to delete the current partition (if you have any pratitions)
    n ...to create a new partition
    p ...to specify it as a PRIMARY partition
    1 ...to set it as the 1ST primary partition
    w ...to write the changes.

Map the disk: (replace ex4TB with your diskname)
```
mkfs.ext4 -L ex4TB /dev/sda1
```

If you have any trouble accessing the disk with "Permission Denied"
Plug in the usb and go to /media/[yourusername] in the terminal.
Type ls -al and see if the USB mount point is owned by root.
If it is, type sudo chown [username]:[username] [mount point]
and enter your password when asked.

## INSTALL Docker:

```
sudo apt update && sudo apt upgrade -y
curl -sSL https://get.docker.com | sh
```

## INSTALL Samba:

```
sudo apt-get install samba samba-common-bin
```
OPTIONAL - If you need to remove old directory (replace ex4TB with your diskname)
```
sudo rm -R "/media/tor/ex4TB/immich"
```
Creating directory (don't use SUDO here) (replace ex4TB with your diskname)
```
mkdir "/media/tor/ex4TB/immich"
mkdir "/media/tor/ex4TB/immich/external"
```
Creating the share configuration-file
```
sudo nano /etc/samba/smb.conf
```
Paste this in the conf file (replace ex4TB with your diskname)
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






Youjhejeje

Credit to [cduff](https://community.spiceworks.com/people/craigduff) at [spiceworks.com](https://community.spiceworks.com/topic/1869414-find-date-in-a-file-name-and-split-it-off) (I used some of his code)

...

After you jkfsdl
This command wilfklsdøfk
```
exift joda jeje
```
jeje
```
jofejklj
```


## Powershell:  "get date from various filenames.ps1"


Youfsdf the script).

Here is a list of filenames that I tested on and it fould all the names containing the dates!
```
2ID_20210704_104414.mp4
VID_20210704_105134.mp4
```
33000 files takes about 60 minutes to run.
