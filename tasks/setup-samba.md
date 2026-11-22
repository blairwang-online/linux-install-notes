# Setup Samba (SMB) file sharing

- [Server side](#server-side-if-this-machine-will-host-shares): this machine hosts shares.
- [Client side](#client-side-if-this-machine-will-connect-to-shares): this machine connects to shares.

A share can live wherever the data is: in your home folder, on an external drive, or anywhere else. On distros with SELinux (AlmaLinux, Fedora), each kind of location needs labelling differently; see [SELinux](#6-selinux-almalinux-and-fedora-only).

## Server side (if this machine will host shares)

### 1. Install

AlmaLinux / Fedora:

```bash
sudo dnf install samba samba-common samba-client policycoreutils-python-utils
```

(`policycoreutils-python-utils` provides `semanage`, used in the SELinux step.)

Debian / Ubuntu:

```bash
sudo apt install samba
```

### 2. Set up your folder

For example, in your home folder:

```bash
mkdir /home/blair/mynetworkdrive
```

For a folder on an external drive, mount the drive first (see [External drive](#external-drive-ext4-xfs-btrfs)), then create the folder on it. Either way, your user needs to own the folder or be able to write to it, because Samba also applies normal Linux permissions.

### 3. Configure the conf file (`/etc/samba/smb.conf`)

```
[global]
...

# Force encryption as per https://serverfault.com/a/1088412
server signing = mandatory
server min protocol = SMB3
server smb encrypt = required

...

[mynetworkdrive]
path = /home/blair/mynetworkdrive
writeable = Yes
browseable = Yes
public = no
valid users = blair
create mask = 0644
directory mask = 0755
```

Remove the _force encryption_ part if you're setting up a server for clients that cannot support SMB encryption for some very special reasons.

Check the file for mistakes with `testparm -s`.

### 4. Setup account

Samba has its own passwords, separate from your login password:

```bash
sudo smbpasswd -a blair
```

### 5. Firewall

AlmaLinux / Fedora: go to Cockpit &rarr; Networking &rarr; Firewall &rarr; Add services (to the correct zone!) &rarr; Filter services to **samba**. Enable **samba** ("TCP: 139, 445; UDP: 138"). Or, on the command line (for the default zone):

```bash
sudo firewall-cmd --permanent --add-service=samba
sudo firewall-cmd --reload
```

Debian / Ubuntu: nothing to do unless ufw is switched on (`sudo ufw status`). If it is:

```bash
sudo ufw allow Samba
```

### 6. SELinux (AlmaLinux and Fedora only)

SELinux only lets Samba read and write files labelled `samba_share_t`. Set the label with `semanage fcontext` followed by `restorecon`, which is permanent. Don't use `chcon`: its labels are temporary and get reset the next time the filesystem is relabelled, which can quietly break the share.

If something doesn't work, look for SELinux denials with `sudo ausearch -m avc -ts recent`.

#### Folder in your home folder

```bash
sudo semanage fcontext -a -t samba_share_t "/home/blair/mynetworkdrive(/.*)?"
sudo restorecon -Rv /home/blair/mynetworkdrive
```

If Samba is still denied access, turn on the boolean that lets Samba into home folders. Note that it covers every user's whole home folder, not just the share:

```bash
sudo setsebool -P samba_enable_home_dirs on
```

#### External drive (ext4, XFS, Btrfs)

These filesystems store SELinux labels on the drive itself. Mount the drive at a fixed path through `/etc/fstab`, rather than letting GNOME mount it under `/run/media/...` (which only happens after you log in, and the path changes if the drive's label does). Find the drive's UUID with `lsblk -f`, then add a line like this to `/etc/fstab` (`nofail` lets the machine boot without the drive):

```
UUID=<uuid> /mnt/external1 ext4 defaults,nofail 0 2
```

Then:

```bash
sudo mkdir -p /mnt/external1
sudo systemctl daemon-reload
sudo mount /mnt/external1
sudo semanage fcontext -a -t samba_share_t "/mnt/external1/mynetworkdrive(/.*)?"
sudo restorecon -Rv /mnt/external1/mynetworkdrive
```

#### External drive (exFAT, FAT, NTFS)

These filesystems can't store SELinux labels, so give the whole drive the label when it's mounted, with the `context=` mount option. They don't store Linux owners either, so set the owner with `uid=` and `gid=` (your user's numbers, from `id`):

```
UUID=<uuid> /mnt/external2 exfat defaults,nofail,uid=1000,gid=1000,context=system_u:object_r:samba_share_t:s0 0 0
```

Then `sudo mkdir -p /mnt/external2`, `sudo systemctl daemon-reload` and `sudo mount /mnt/external2`. Use `ntfs3` or `vfat` instead of `exfat` for those filesystems.

If an NTFS drive is mounted with `ntfs-3g` (FUSE) rather than `ntfs3`, `context=` doesn't take effect. Turn on this boolean instead:

```bash
sudo setsebool -P samba_share_fusefs on
```

#### Last resort

`samba_export_all_rw` lets Samba read and write any file, whatever its label. It works everywhere but gives up SELinux's protection for Samba, so only use it if nothing above works:

```bash
sudo setsebool -P samba_export_all_rw on
```

### 7. Start Samba

AlmaLinux / Fedora (the service is called `smb`):

```bash
sudo systemctl enable smb
sudo systemctl restart smb
sudo systemctl status smb
```

Debian / Ubuntu (the service is called `smbd`, and it's already running after install):

```bash
sudo systemctl restart smbd
sudo systemctl status smbd
```

Restart the service after any change to `smb.conf`.

... See also https://docs.fedoraproject.org/en-US/quick-docs/samba/

## Client side (if this machine will connect to shares)

Install the tools:

```bash
sudo dnf install cifs-utils   # AlmaLinux / Fedora
sudo apt install cifs-utils   # Debian / Ubuntu
```

Some instructions use `/mnt/` instead of `/media/`. The main difference as far as I am concerned is that using `/media/` makes it appear as a removable drive in Nautilus file manager. For more information please refer to https://askubuntu.com/questions/22215/why-have-both-mnt-and-media

### Option 1: mount from the command line (any distro)

```bash
sudo mkdir /media/myshare1
sudo mkdir /media/myshare2

sudo mount -t cifs -o rw,username=blair,nounix,dir_mode=0777,file_mode=0777 //server1/myshare1 /media/myshare1 # mount myshare1
sudo mount -t cifs -o rw,username=blair,nounix,dir_mode=0777,file_mode=0777 //server1/myshare2 /media/myshare2 # mount myshare2

sudo umount /media/myshare1
sudo umount /media/myshare2
```

To avoid typing the password each time, replace `username=blair` with `credentials=/home/blair/myshare-credentials.txt` (see the credentials file format in option 2).

This is the method to use on **Fedora Silverblue**. Don't use the fstab method there: fstab is writeable, but you cannot chmod `/bin/mount`, so the method below doesn't work.

### Option 2: fstab, so you can mount without sudo (AlmaLinux, Fedora)

First, `mkdir /media/myshare1`, then:

Add to `/etc/fstab`:

```
//myserver/myshare1 /media/myshare1 cifs credentials=/home/myusername/myshare-credentials.txt,noauto,user 0 0
```

(I found that I had to specify `/home/myusername/` instead of `~/`)

Credentials file formatted like so:

```
username=blair
password=mypasswordgoeshere
```

It holds a password, so make it readable only by you: `chmod 600 /home/myusername/myshare-credentials.txt`.

Then as per https://superuser.com/questions/1444883/user-cifs-mounts-not-supported-fedora-30, we need to set the suid bit:

```bash
sudo chmod u+s /bin/mount
sudo chmod u+s /bin/umount
sudo chmod u+s /usr/sbin/mount.cifs
```

Then need to refresh systemd to use the updated fstab:

```bash
sudo systemctl daemon-reload
```

Then can mount like:

```bash
mount //myserver/myshare1
```

To check the properties of the Samba share, run `mount -t cifs`.
