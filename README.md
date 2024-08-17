# Purpose
I have a macmini only have 256G SSD and meanwhile have a NAS with 30T, Find some way to automount NAS NFS to macmini for extend HDD size

# Research
As we know macos is from Unix, Unix system basically all support NFS, after research find that macos support automount, master file location is /etc/auto_master

Considering the network latency and bandwidth, I will not put any small files into automount folder

Here is my plan

1. Create NFS share on NAS
2. Create a mount point on macos
3. Add nfs auto mount define file on macos
4. Add nfs auto mount define file to auto_master

# Implement

Skip NFS create step

Create a mount point for NFS on macos
```
mkdir -p /System/Volumes/Data/nfs
```

For now I have these NFS mount point on my NAS

```
% showmount -d 192.168.3.240
Directories on 192.168.3.240:
/volume1/BaiduPan
/volume1/MacMiniData
/volume1/Movies
/volume1/Music
/volume1/Shares
/volume1/roms
```

create /etc/auto_nfs 
```
data -fstype=nfs,soft,resvport,rw,noowners 192.168.3.240:/volume1/MacMiniData
movies -fstype=nfs,soft,resvport,noowners 192.168.3.240:/volume1/Movies
music -fstype=nfs,soft,resvport,noowners 192.168.3.240:/volume1/Music
```

add /etc/auto_nfs to auto_master
```
...
/System/Volumes/Data/nfs auto_nfs
```

last active automount
```
automount -vc
```

# Issues
Sometime(Maybe system update) auto_master will lost auto_nfs define, please add back manually

or use this script to check regularly

```
#!/bin/sh

[ $UID -ne 0 ] && echo "require root permission" && exit 1

if [ $(grep '^/System/Volumes/Data/nfs auto_nfs' -c /etc/auto_master) -eq 0 ];then
    echo '/System/Volumes/Data/nfs auto_nfs' >> /etc/auto_master && automount -vc
fi
```

