# proxmox-install-guide-and-maintenance
Proxmox install guide and maintenance.

## Making use of the entire disk after installation of your Proxmox Backup Server
```
vgdisplay
lvextend -l +100%FREE /dev/mapper/pbs/root
resize2fs /dev/mapper/pbs/root
```

## Remove Enterprise Repo and Nag Box on logon
https://github.com/CarmineCodes/Proxmox-No-Subscription-No-Problem

## Netgear ReadyNAS Duo V1 as DataStore
### Proxmox Backup Server changes
Add to [global] section of /etc/samba/smb.conf
```
client min protocol = CORE
client max protocol = SMB3
```
### ReadyNAS Duo changes
NFS protocol must be checked.
![Screenshot 2024-04-07 180843](https://github.com/ejbaling/proxmox-install-and-maintenance/assets/3113782/dae1f6a9-d01e-483a-b888-9cd967b14cae)

To get a list of shares available on a host
```
smbclient -L <ip address> -U <username>
```
List of shares example:

![Screenshot 2024-04-07 180843](https://github.com/ejbaling/proxmox-install-and-maintenance/assets/3113782/999a4c8d-b6a8-4c00-b004-d29b134c73e7)

### Mount an SMB Share and Create a Datastore using this path.
```
mkdir /mnt/smb-share
mount -t cifs //<server-ip>/<share-path> /<mount-point> -o vers=1.0,user=<username>
Example: mount -t cifs //10.0.0.5/backup /smb-share -o vers=1.0,user=emmanuel
```
To check the mount
```
mount -t cifs
```
In your Proxmox Backup Server UI, click Add Datastore then follow the on-screen steps.  
Datastore creation could take a long time to complete, 30% progress is around 1 hour.
