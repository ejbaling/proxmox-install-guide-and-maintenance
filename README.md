# proxmox-install-guide-and-maintenance
Proxmox install guide and maintenance.

## Making use of the entire disk after installation of your Proxmox Backup Server
```
vgdisplay
lvextend -l +100%FREE /dev/mapper/pbs-root
resize2fs /dev/mapper/pbs-root
```

## Making use of the entire disk after installation of your VM's
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
Restart the service.
### ReadyNAS Duo changes
NFS protocol must be checked.
![Screenshot 2024-04-07 180843](https://github.com/ejbaling/proxmox-install-and-maintenance/assets/3113782/dae1f6a9-d01e-483a-b888-9cd967b14cae)

To get a list of shares available on a host
```
smbclient -L <ip address> -U <username>
```
List of shares example:

![Screenshot 2024-04-07 180843](https://github.com/ejbaling/proxmox-install-and-maintenance/assets/3113782/999a4c8d-b6a8-4c00-b004-d29b134c73e7)

If you get the following error
```
Protocol negotiation to server 10.0.0.5 (for a protocol between SMB2_02 and SMB3) failed: NT_STATUS_CONNECTION_DISCONNECTED
```
Update smb.conf, see above. Don't forget to install cifs-utils `sudo apt-get install cifs-utils`
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
This is how it looks like

 ![Screenshot 2024-08-18 132914](https://github.com/user-attachments/assets/c39eb7c3-e3e3-45c8-b76a-cac3b204b230)

Issue this command to check. If there are no errors then is is OK.
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

## TrueNAS Scale as Storage
https://www.youtube.com/watch?v=VlqMVtuoPP8&t=800s

## Add new Hard Disk
https://www.vinchin.com/vm-tips/proxmox-mount-disk.html

Check the device name of the hard drive to be added.
![Screenshot 2025-01-25 233715](https://github.com/user-attachments/assets/659863f5-bb62-4633-94ad-c995393c88c2)

Open a node shell and input the command:
```
fdisk /dev/sdb
```

Enter "d" to delete partions. Enter "n" to create a partition. Enter "w" to execute the opertion.

Format the new partition:
```
mkfs -t ext4 /dev/sdb1
```

From this Youtube video https://youtu.be/U-UTMuhmC1U?si=t6fypsK7HUAqVNFi

Enter the following command to get the device Id:
```
ls -n /dev/disk/by-id/
```
![Screenshot 2025-01-26 002459](https://github.com/user-attachments/assets/3054cd30-2596-45b8-a122-47c699e62666)

Enter the following comand to pass thru disk to your VM:
```
/sbin/qm set 109 -virtio2 /dev/disk/by-id/ata-SAMSUNG_HD321HJ_S2GVJ9AZ813210
```
![Screenshot 2025-01-26 021927](https://github.com/user-attachments/assets/d06db27c-cd07-418d-a01c-1cc10c1d24e6)

Make sure the checkbox next to the Backup is de-selected.
![Screenshot 2025-01-26 022131](https://github.com/user-attachments/assets/044d2b52-ca95-463a-b78f-84578049f4bc)
