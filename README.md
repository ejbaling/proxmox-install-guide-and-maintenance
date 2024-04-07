# proxmox-install-and-maintenance
Proxmox install and maintenance.

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
