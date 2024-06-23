# proxmox-install-guide-and-maintenance
Proxmox install guide and maintenance.

## Making use of the entire disk after installation of your Proxmox Backup Server
```
vgdisplay
lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
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
Add samba user credentials.
```
nano /etc/samba/.smbcreds
```
Set username and password.
```
username=smb-username
password=smb-password
```
Save file and make it readable by root only.
```
chmod 400 /etc/samba/.smbcred
```
Auto-mount on boot by adding the following to /etc/fstab
```
//10.0.0.5/proxmox-backup /mnt/smb-share cifs credentials=/etc/samba/.smbcred,vers=1.0 0 0
```
Issue this command to check
```
mount -a
```
In your Proxmox Backup Server UI, click Add Datastore then follow the on-screen steps.  
Datastore creation could take a long time to complete, 30% progress is around 1 hour. This fails me twice.

Use the CLI to create the datastore
```
proxmox-backup-manager datastore create proxmox-backup /mnt/smb-share
```
## External USB Device as Datastore
### Create a Backup Operator User in Proxmox Backup Server
https://4sysops.com/archives/proxmox-backup-server-install-and-configure/
