# AlmaLinux setup notes by Blair

## Rationale

I'd previously favoured Fedora (see [fedorasetup](https://github.com/blairw/fedorasetup)) for Linux purposes, but I'd found that it still moves relatively too quickly. Maybe it's because I'm now quite satisified with the featureset of something circa Fedora 34 (2021), which is good enough for most server and workstation use cases. In the past I went all the way to Arch Linux just to get all the newest features that I wanted...

AlmaLinux is now the de facto stable downstream distro based on Fedora, or at least [CERN seems to feel that way](https://linux.web.cern.ch/almalinux/). If it's good enough for particle physics then I'm certainly receptive of it. 🚀

## About these Setup Notes

Since this document is mostly for my own records, unless you are me, not everything below is certainly relevant for you. But maybe some of it is / maybe this might help someone else trying to get something working? 😊

These notes started out on AlmaLinux 9. Sections and notes marked **EL10** are for AlmaLinux 10, which is what I now use for new installs, including a [workstation/server hybrid](#workstationserver-hybrid-el10) setup. The EL10 material was drafted with AI assistance and hasn't all been tested end-to-end yet, so treat it with care.

## Install AlmaLinux

- Complete tasks as per [install-to-bare-metal.md](../tasks/install-to-bare-metal.md)
- Complete tasks as per [setup-preferred-shell.md](../tasks/setup-preferred-shell.md)
- Complete tasks as per [gnome-configurations.md](../tasks/gnome-configurations.md)

### Installing AlmaLinux 10 (EL10)

Which ISO:

- Use the **DVD** ISO (about 9 GB, fine on a 32 GB USB stick). It installs offline and offers every environment, including **Server with GUI**, plus LUKS encryption in the installer.
    - _Boot_ is the same installer but downloads packages, so it needs a working network during the install. _Minimal_ has no GUI. _Live GNOME_ copies a desktop image onto the disk, with little control over what gets installed.
- Use the standard `x86_64` ISO if the CPU supports x86-64-v3 (roughly: AVX2, so Intel Haswell or later, including Intel N100/N150). For older CPUs, use AlmaLinux's `x86_64_v2` build.
- Check the SHA-256, then write the ISO with `dd` or Fedora Media Writer.

In the installer:

- **Keyboard:** set the right layout. The LUKS passphrase prompt at boot uses it.
- **Installation Destination:** tick only the disk you're installing to (not the USB stick, and not any other internal drive). Tick **Encrypt my data** (LUKS2). For a server, choose _Custom_, let the installer create the partitions automatically, then adjust: `/` about 100 GiB, swap about 4 GiB, and the rest as `/srv` rather than `/home`.
    - Btrfs has been offered since AlmaLinux 10.1. It's an AlmaLinux addition rather than standard RHEL, so XFS on LVM is the safe default; Btrfs is worth it if you want snapshots.
- **Network & Host Name:** set the permanent hostname now (Forgejo's URLs will use it), and connect Wi-Fi and/or Ethernet so the connection profiles are saved.
- **Software Selection:** _Workstation_ for a workstation. _Server with GUI_ for a server or [hybrid](#workstationserver-hybrid-el10), adding _Headless Management_ (Cockpit) and _File and Storage Server_ (Samba) if they're offered.
- **KDUMP:** turn it off on machines with little RAM, since it reserves memory for crash dumps.
- **User:** create your user and tick _Make this user administrator_. Leave root locked.

## Update everything

**Caution:** If intending to use Virtualbox, you may need to follow the `--exclude=kernel*` instructions based on [fedorasetup](https://github.com/blairw/fedorasetup), rather than the update-all approach documneted here.

That said:

First, set `fastestmirror=True` in `/etc/dnf/dnf.conf` as per https://discussion.fedoraproject.org/t/how-to-change-default-dnf-server-country/73107/4?u=ankursinha

Next, run the tasks - note that `noglob` is required for zsh as per https://github.com/ohmyzsh/ohmyzsh/issues/31.

```zsh
# Update everything
noglob sudo dnf update -y

# Refresh the cache
sudo dnf makecache --refresh
```

## Setup dnf with RPMFusion and EPEL (workstation + server)

NOTE: This is required for server, even for basic packages like `aria2`...

WARNING: If copy-pasting the below commands into iTerm2 on Mac, be careful that some escapes (`\`) get copied in unnecessarily. You will need to manually remove these inside the Terminal.

```zsh
# Get RPMFusion and EPEL together
sudo dnf install --nogpgcheck https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(rpm -E %rhel).noarch.rpm
sudo dnf install --nogpgcheck https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-$(rpm -E %rhel).noarch.rpm https://mirrors.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-$(rpm -E %rhel).noarch.rpm
```

Next steps:

```zsh
# Refresh the cache again...
sudo dnf makecache --refresh

# Then do this one, which forces a bit of a cleanup
sudo dnf install aria2
```

Workstation only (!):

```zsh
sudo dnf install audacity ffmpeg obs-studio vlc
sudo dnf install geany geany-plugins-addons geany-plugins-autoclose geany-plugins-automark geany-plugins-codenav geany-plugins-git-changebar geany-plugins-geanyprj geany-plugins-lineoperations geany-plugins-markdown geany-plugins-overview geany-plugins-projectorganizer geany-plugins-treebrowser geany-plugins-workbench
```

Adapted from https://rpmfusion.org/Configuration/

Use this method of installing EPEL from the Fedora project rather than the one in AlmaLinux, this will give us a newer version.

NOTE: If you're told that the update(s) above have installed a new kernel, reboot now.

## Setup dnf with CRB (workstation only)

CRB is needed for the R programming language and probably some other useful stuff too...

**EL10:** enable CRB on servers too (the first two commands below, not `R`), since many EPEL 10 packages depend on it.

```zsh
sudo dnf config-manager --set-enabled crb
sudo dnf makecache --refresh
sudo dnf install R
```

## Install useful packages

```zsh
# Common install for workstation + server:
noglob sudo dnf install -y fastfetch git-lfs htop nano screen npm

# If this machine is a workstation, also install:
noglob sudo dnf install -y chromium gedit-plugin-textsize gimp gnome-extensions-app gnome-tweaks libreoffice nextcloud-client-nautilus overpass-fonts
```

## Setup Cockpit

```zsh
sudo dnf install cockpit
sudo systemctl enable --now cockpit.socket
sudo firewall-cmd --add-service=cockpit
sudo firewall-cmd --add-service=cockpit --permanent
```

Adapted from https://cockpit-project.org/running

Once set up, go to Cockpit:

- Overview &rarr; View metrics and history &rarr; **Install cockpit-pcp**
- Then logout, log back in
- Overview &rarr; View metrics and history &rarr; **Metrics settings** and enable **Collect metrics** but maybe not Export to network (!).
- Applications &rarr; Machines &rarr; Install
    - **EL10 server:** skip Machines on a low-RAM machine. Install _Podman containers_ (`cockpit-podman`) instead, which is handy for [Forgejo](../tasks/forgejo-podman.md).

## Setup Dropbox

Get this started sooner rather than later, so that it can start fetching your files in the background.

```zsh
# Install
cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -

# Get Dropbox py
mkdir -p ~/00blair/dropboxpy
wget -O ~/00blair/dropboxpy/dropbox.py https://www.dropbox.com/download\?dl\=packages/dropbox.py
chmod +x ~/00blair/dropboxpy/dropbox.py 

# Run
~/.dropbox-dist/dropboxd

# Setup exclusions
~/00blair/dropboxpy/dropbox.py exclude add ~/Dropbox/blair-extras2
~/00blair/dropboxpy/dropbox.py exclude add ~/Dropbox/blair-extras3
~/00blair/dropboxpy/dropbox.py exclude add ~/Dropbox/blair-extras4
```

You will need to configure firewall to enable LAN sync properly:

```zsh
# get zone name e.g. `public`, `FedoraServer`
sudo firewall-cmd --get-active-zones

# confirm that dropbox-lansync is available
sudo firewall-cmd --get-services | grep dropbox

# enable, zone=public
sudo firewall-cmd --zone=public --add-service=dropbox-lansync --permanent

# enable, zone=FedoraServer
sudo firewall-cmd --zone=FedoraServer --add-service=dropbox-lansync --permanent

# reload
sudo firewall-cmd --reload
```

I use this version because it works across server and workstation environments, across desktop environments, with no dependency on package managers and so on.


## Samba (SMB) - server-side configuration (if this machine will host shares)

First, install the necessary packages:

```zsh
sudo dnf install samba samba-common samba-client
```

Next, setup your folder:

```zsh
mkdir /home/blair/mynetworkdrive
```

Then, configure the conf file (`/etc/samba/smb.conf`):

```markdown
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
write list = user
```

Remove the _force encryption_ part if you're setting up a server for clients that cannot support SMB encryption for some very special reasons.

Next, go to Cockpit &rarr; Networking &rarr; Firewall &rarr; Add services (to the correct zone!) &rarr; Filter services to **samba**. Enable **samba** ("TCP: 139, 445; UDP: 138").

Finally:

```zsh

# Setup account
sudo smbpasswd -a blair

# Set correct folder permissions
sudo semanage fcontext -a -t public_content_t "/home/blair/mynetworkdrive(/.*)?"

# Activate correct folder permissions
sudo restorecon -R /home/blair/mynetworkdrive
chcon -t samba_share_t /home/blair/mynetworkdrive -R

sudo setsebool -P samba_domain_controller on
sudo setsebool -P samba_enable_home_dirs on
```

Note: systemctl commands...

```zsh
sudo systemctl enable smb
sudo systemctl restart smb
sudo systemctl status smb
```

... See also https://docs.fedoraproject.org/en-US/quick-docs/samba/

### EL10 server: shares under /srv

On a dedicated server, I now keep shares under `/srv` instead of `/home`. That keeps the SELinux setup simple and keeps the data away from GNOME's file indexer. Not tested end-to-end yet.

```zsh
sudo dnf install -y samba samba-client policycoreutils-python-utils
sudo mkdir -p /srv/samba/mynetworkdrive
sudo chown blair: /srv/samba/mynetworkdrive
sudo semanage fcontext -a -t samba_share_t "/srv/samba(/.*)?"
sudo restorecon -Rv /srv/samba
```

Use the same `[global]` settings as above, with a share like this:

```markdown
[mynetworkdrive]
path = /srv/samba/mynetworkdrive
writeable = Yes
browseable = Yes
public = no
valid users = blair
create mask = 0644
directory mask = 0755
```

Then:

```zsh
sudo smbpasswd -a blair
sudo firewall-cmd --permanent --add-service=samba
sudo firewall-cmd --reload
sudo systemctl enable --now smb
```

The `samba_domain_controller` and `samba_enable_home_dirs` booleans from the section above aren't needed for shares under `/srv`.


## Samba (SMB) - client-side configuration (if this machine will connect to shares)

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

Then as per https://superuser.com/questions/1444883/user-cifs-mounts-not-supported-fedora-30, we need to set the suid bit:

```zsh
sudo chmod u+s /bin/mount
sudo chmod u+s /bin/umount
sudo chmod u+s /usr/sbin/mount.cifs
```

Then need to refresh systemd to use the updated fstab:

```zsh
sudo systemctl daemon-reload
```

Then can mount like:

```zsh
mount //myserver/myshare1
```

To check the properties of the Samba share, run `mount -t cifs`.




## Git / GitHub credentials saver

- Complete tasks as per [git-and-github-configuration.md](../tasks/git-and-github-configuration.md)

## Install Snaps

Adapted from https://www.liquidweb.com/kb/how-to-install-snapcraft-on-ubuntu-centos/

```zsh
sudo dnf install -y snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap

# Close Terminal and open it again here

sudo snap install lolcat
```

## Install Flatpaks

Download the flatpak file and then just double-click it, it will prompt you to install using a little GUI.

- Handbrake: https://handbrake.fr/downloads.php and go to Flatpak section


## Install from GNOME Software Center

- Dialect
- gitg
- Do NOT use this for: Dropbox


## Fix rendering of some fonts (e.g., Monaco)

https://superuser.com/questions/93924/monaco-font-not-antialiased-in-some-gtk-apps


## Install some software manually, if needed

- Brave: https://brave.com/linux/#fedora-rockyrhel
- FSearch: https://copr.fedorainfracloud.org/coprs/cboxdoerfer/fsearch/
- RStudio: https://posit.co/download/rstudio-desktop/
- VSCodium: https://vscodium.com/#install-on-fedora-rhel-centos-rockylinux-opensuse-rpm-package


## Pasaffe

PATH needs to include `/home/blair/.local/bin` but otherwise confirmed working! pasaffe-0.58-1 on AlmaLinux 9.3 (python --version == Python 3.9.18).

**EL10:** ships Python 3.12. Not tried there yet.

Adapted from https://answers.launchpad.net/pasaffe/+question/658239

1. Download latest tar.gz from https://launchpad.net/pasaffe
2. `sudo dnf install -y rpmdevtools python3-pip python3-distutils-extra`
3. `pip3 install unidecode`
4. Navigate to the folder where the latest tar.gz was downloaded
5. `tar xvzf pasaffe-0.58.tar.gz` (replace with equivalent filename)
6. Navigate to extracted folder
7. `python setup.py bdist_rpm`
8. `find dist/*.rpm`
9. `sudo dnf install -y dist/pasaffe-0.58-1.noarch.rpm`
10. If needed, symlink an existing psafe3 database to `~/.local/share/pasaffe/pasaffe.psafe3`


## Setup folder for custom fonts

```zsh
mkdir ~/.local/share/fonts
# then copy whatever fonts you want
fc-cache -fv
```

## Replicate pbcopy

- Complete tasks as per [replicate-pbcopy.md](../tasks/replicate-pbcopy.md)

## GNOME Extensions

- Complete tasks as per [gnome-extensions.md](../tasks/gnome-extensions.md)

## GNOME prevent sleep-if-no-one-logs-in

Adapted from https://askubuntu.com/a/1132068

Not tested yet, might need `sudo dnf install dbus-x11`, TODO: to be tested...

```
blair@fedoraworkstation ~
❯ sudo -u gdm dbus-launch gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type
'suspend'

blair@fedoraworkstation ~
❯ sudo -u gdm dbus-launch gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type "nothing"

blair@fedoraworkstation ~
❯ sudo -u gdm dbus-launch gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type
'nothing'
```

**EL10:** X11 is gone, so `dbus-x11` isn't the answer there. On an always-on machine, mask the sleep targets instead, as in [Never suspend](#never-suspend) below.

## Workstation/server hybrid (EL10)

For an always-on machine that is mainly a server (Samba, Forgejo) but keeps GNOME for the odd time a display is plugged in. Install with **Server with GUI** (see [Installing AlmaLinux 10](#installing-almalinux-10-el10)). For an Intel N100-class mini PC, also complete tasks as per [intel-n100-server.md](../tasks/intel-n100-server.md).

### Desktop choice on EL10

RHEL 10, and so AlmaLinux 10, is Wayland-only and has dropped the X11 server. GNOME is the default and KDE Plasma is in EPEL. As of late 2026, Xfce and MATE aren't in EPEL 10 and only exist in unofficial COPR repositories, which I'd avoid on a server.

### Boot to the console, start GNOME only when needed

An installed desktop costs nothing but disk space until it runs.

```zsh
sudo systemctl set-default multi-user.target
```

When a display is plugged in, log in on the console and run `sudo systemctl start gdm`. To go back to booting straight into GNOME: `sudo systemctl set-default graphical.target`.

### Never suspend

```zsh
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

This stops GNOME and GDM from suspending the machine, whatever their own settings say.

### Updates: no PackageKit, security updates via dnf-automatic

Mask PackageKit so GNOME Software doesn't check for and download updates in the background (GNOME Software can't install anything while it's masked), and let `dnf-automatic` apply security updates:

```zsh
sudo systemctl mask packagekit.service
sudo dnf install -y dnf-automatic
```

In `/etc/dnf/automatic.conf`, under `[commands]`, set `upgrade_type = security` and `apply_updates = yes`. Then:

```zsh
sudo systemctl enable --now dnf-automatic.timer
```

### Keep shared data out of /home

Put Samba shares and Forgejo data under `/srv` (see [shares under /srv](#el10-server-shares-under-srv) and [forgejo-podman.md](../tasks/forgejo-podman.md)). Besides keeping SELinux simple, this means GNOME's file indexer (localsearch) won't crawl them.

### LUKS: unlock with TPM + PIN, and back up the header

Needs the firmware TPM switched on in the BIOS (Intel PTT or AMD fTPM). The passphrase set during install stays as a fallback. Not tested yet.

```zsh
# Find the LUKS partition (FSTYPE crypto_LUKS)
lsblk -f

# Enrol the TPM with a PIN (replace the device with yours)
sudo systemd-cryptenroll --tpm2-device=auto --tpm2-with-pin=yes /dev/nvme0n1p3
```

Then add `tpm2-device=auto` to that device's options in `/etc/crypttab` (the fourth field, comma-separated; if it says `none`, replace it), rebuild the initramfs with `sudo dracut -f`, and reboot. It should now ask for the PIN rather than the passphrase.

BIOS or firmware updates, or Secure Boot changes, can stop the TPM unlocking. If that happens, enter the passphrase, then re-enrol:

```zsh
sudo systemd-cryptenroll --wipe-slot=tpm2 --tpm2-device=auto --tpm2-with-pin=yes /dev/nvme0n1p3
```

Back up the LUKS header, and keep the file somewhere other than this machine:

```zsh
sudo cryptsetup luksHeaderBackup /dev/nvme0n1p3 --header-backup-file ~/luks-header-$(hostname).img
```

### SSD TRIM

```zsh
sudo systemctl enable --now fstrim.timer
```

For TRIM to reach the SSD through LUKS, the device's options in `/etc/crypttab` need to include `discard` (then `sudo dracut -f`). The trade-off is that it reveals which blocks are unused, which I'm happy to accept.

### Power profile

```zsh
sudo tuned-adm profile powersave
tuned-adm active
```

Switch back to `balanced` if anything (USB devices, for example) misbehaves.

### Forgejo

- Complete tasks as per [forgejo-podman.md](../tasks/forgejo-podman.md)

## Ongoing Maintenance

### Checking NTP

See https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/configuring-time-synchronization_configuring-basic-system-settings#using-chrony_configuring-time-synchronization

### Using ddcutil for the equivalent of Mac's MonitorControl (set screen brightness)

_Adapted from https://www.reddit.com/r/raspberry_pi/comments/10o2nzt/comment/j6c9q4h/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button_

You might be prompted to install `ddcutil`.

To get the current brightness setting:

```zsh
sudo ddcutil getvcp 0x10
```

To set the brightness to, for example, 81%:

```zsh
sudo ddcutil setvcp 0x10 81
```

Be careful with DDC, not every monitor supports it, though most modern monitors do.

