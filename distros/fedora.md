# Fedora Linux setup notes by Blair

Update: This document is no longer maintained, for Linux purposes I've switched over to Almalinux. See https://github.com/blairw/almalinuxsetup

## Rationale

This is mostly written for my own notes so that I remember why I did this.

### Rationale for Linux as a workstation environment

To be honest, my preferred workstation environment is actually macOS. I also sometimes need to use Windows for very specific tasks involving software that only works properly on Windows.

However, what I have learnt over years of working in tech (including tech-related academic work) is that it is useful to challenge myself to work entirely in Linux at least somewhat regularly. From a skills perspective, working in Linux forces me to stay competent with *nix-based sysadmin, bash/zsh, Python, etc. From a workflow perspective, working in Linux acts as a mild but non-trivial deterrant against overreliance on commercial software packages. I have nothing against commercial software packages but in general find that most of them are prone to at least one of the following:

- Ongoing licensing costs
- Planned obsolescence
- Increasing slowness (cf. [Wirth's Law](https://en.wikipedia.org/wiki/Wirth%27s_law))

In practice, I will probably rely on a [bricolage](https://en.wikipedia.org/wiki/Bricolage#Information_systems) of web-based, Mac, Windows and Linux solutions.

### Rationale for Fedora Linux in particular

Having tried, at various stages over the past decade, a range of distros (incl. Ubuntu, Debian, elementaryOS, Linux Mint, Arch, Antergos, OpenSUSE, CentOS, Oracle Linux, RHEL), I keep coming back to Fedora because of a few very specific things:

1. The Fedora installer automates the setup of LUKS **full-disk encryption** and the boot screen where you enter your disk password which is clean and elegant. (Full-disk encryption is very important to me due to the sensitive data that I often work with.)

2. Understanding the overall sysadmin process is useful due to the **market dominance of RHEL and rpm-based derivatives** like CentOS / Oracle Linux / etc. This might be a bit of a question mark now with the CentOS Stream controversy but RHEL seems like it is so entrenched that it won't go away too soon. Arguably deb-based distros like Debian/Ubuntu are increasingly more relevant (crucially, Ubuntu is free on WSL, Fedora is not); but then I miss out on the above point re LUKS setup.

3. It's somehow **both cutting-edge and stable**. Arch is more cutting-edge but I had a few issues with upgrades breaking things and not being able to find a fix since the rolling release model means that I can't find specific advice like "fedora 33 luks btfrs swap not working" because the concept of a "version 33" doesn't exist. Debian/CentOS/RHEL are more stable but then a lot of handy new stuff doesn't work.

4. Fedora **MATE** is set up nicely. I use MATE (fork of GNOME 2) as basically a solution to Wirth's Law. Modern OS GUIs (macOS, Windows, even GNOME 3) are all subject to Wirth's Law, they're not as lean as they could be because they assume you have decent specs. I have decent specs but still would prefer speed over whatever fancy things are in macOS / Windows / GNOME 3. I was using GNOME 2 in 2008 on a budget Dell PC in my parents' apartment running with a slow IDE hard drive and a Pentium 4. It was fast even on that; it's lightning fast on my 2019 budget Acer laptop with an NVMe SSD and a Core i5.

- **Hibernate actually works!** To be honest I almost gave up on Fedora because getting Hibernate to work on the wacky combination of LUKS, btrfs and zram replacing swap was a pain. But apparently (cf. point 2), enough people on Fedora care about Hibernate that someone else already solved it (:

---

## Setup tasks

Since this document is mostly for my own records, unless you are me, not everything below is certainly relevant for you. But maybe some of it is / maybe this might help someone else trying to get something working? (:

See also: https://mutschler.eu/linux/install-guides/fedora-post-install/ (similar endeavour, also by a Business School academic who likes looking into Linux!)

### Installing Fedora

- Complete tasks as per [install-to-bare-metal.md](../tasks/install-to-bare-metal.md)
- Complete tasks as per [setup-preferred-shell.md](../tasks/setup-preferred-shell.md)
- Complete tasks as per [gnome-configurations.md](../tasks/gnome-configurations.md)

### Get the latest updates and get useful packages

First, set `fastestmirror=True` in `/etc/dnf/dnf.conf` as per https://discussion.fedoraproject.org/t/how-to-change-default-dnf-server-country/73107/4?u=ankursinha

Next, run the tasks - note that `noglob` is required for zsh as per https://github.com/ohmyzsh/ohmyzsh/issues/31.

```zsh
# Update everything except the kernel (be careful with kernel upgrades!):
noglob sudo dnf update -y --exclude=kernel*

# Common install for workstation + server:
noglob sudo dnf install -y aria2 htop lolcat nano neofetch screen

# If this machine is a workstation:
noglob sudo dnf install -y audacity geany geany-themes gimp
```

Some notes:
- If using MATE, get command-line `redshift` because `redshift-gtk` doesn't seem to do what it is supposed to (no tray icon, runs in background). Not required for GNOME which has a night mode built in.
- If using MATE, get `synapse` as a Spotlight kind of app searcher. Not required for GNOME which has the Activities mode.
- Added `--exclude=kernel*` as per https://ask.fedoraproject.org/t/howto-prevent-kernel-updates/3001/2 because I had a situation in which `kernel` was available at `5.11.13-200` but `kernel-headers` was only available at `5.11.11-200` which prevented me from patching the kernel for VirtualBox.

	```
	❯ sudo dnf list installed | grep kernel
	abrt-addon-kerneloops.x86_64                2.14.5-1.fc33                          @updates                             
	kernel.x86_64                               5.8.15-301.fc33                        @anaconda                            
	kernel.x86_64                               5.11.12-200.fc33                       @updates                             
	kernel.x86_64                               5.11.13-200.fc33                       @updates                             
	kernel-core.x86_64                          5.8.15-301.fc33                        @anaconda                            
	kernel-core.x86_64                          5.11.12-200.fc33                       @updates                             
	kernel-core.x86_64                          5.11.13-200.fc33                       @updates                             
	kernel-devel.x86_64                         5.11.13-200.fc33                       @updates                             
	kernel-headers.x86_64                       5.11.11-200.fc33                       @updates                             
	kernel-modules.x86_64                       5.8.15-301.fc33                        @anaconda                            
	kernel-modules.x86_64                       5.11.12-200.fc33                       @updates                             
	kernel-modules.x86_64                       5.11.13-200.fc33                       @updates                             
	kernel-modules-extra.x86_64                 5.8.15-301.fc33                        @anaconda                            
	kernel-modules-extra.x86_64                 5.11.12-200.fc33                       @updates                             
	kernel-modules-extra.x86_64                 5.11.13-200.fc33                       @updates                             
	kernel-srpm-macros.noarch                   1.0-3.fc33                             @fedora                              
	kernel-tools-libs.x86_64                    5.11.11-200.fc33                       @updates                             
	libreport-plugin-kerneloops.x86_64          2.14.0-15.fc33                         @updates
	```


### Setup Cockpit

Follow the instructions at https://cockpit-project.org/running

### Setup Dropbox

Get this started first so that it can start fetching your files in the background.

For Workstation: Get the rpm from https://www.dropbox.com/install-linux

For Server:

```zsh
# Install
cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -

# Get Dropbox py
mkdir -p ~/00blair/dropboxpy
wget -O ~/00blair/dropboxpy/dropbox.py https://www.dropbox.com/download\?dl\=packages/dropbox.py

# Run
~/.dropbox-dist/dropboxd

# Setup exclusions
~/00blair/dropboxpy/dropbox.py exclude add ~/Dropbox/blair-extras2
~/00blair/dropboxpy/dropbox.py exclude add ~/Dropbox/blair-extras3
~/00blair/dropboxpy/dropbox.py exclude add ~/Dropbox/blair-extras4
```

For both Workstation and Server, you will need to configure firewall to enable LAN sync properly:

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

### Git / GitHub credentials saver

- Complete tasks as per [git-github-credentials-saver.md](../tasks/git-github-credentials-saver.md)

### Fix Monaco rendering

https://superuser.com/questions/93924/monaco-font-not-antialiased-in-some-gtk-apps


### Install some software manually (Dropbox etc)
- https://computingforgeeks.com/how-to-install-netbeans-ide-on-fedora/
- https://www.microsoftedgeinsider.com/en-us/download?platform=linux-rpm
- https://code.visualstudio.com/docs/setup/linux
- https://linuxhint.com/install-microsoft-teams-fedora/
- https://slack.com/intl/en-au/downloads/instructions/fedora
- https://github.com/ramboxapp/community-edition/releases/
- https://zoom.us/download?os=linux
- https://copr.fedorainfracloud.org/coprs/dusansimic/caprine/



### Install Spotify via Flatpak
As per https://docs.fedoraproject.org/en-US/quick-docs/installing-spotify/ - I am using the Flatpak option because I found that `lpf-spotify-client` was not building on my machine.

```zsh
sudo dnf install -y flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
sudo flatpak install flathub com.spotify.Client
```

### FSearch

https://copr.fedorainfracloud.org/coprs/cboxdoerfer/fsearch/

### Wine

Instructions adapted from https://computingforgeeks.com/how-to-install-wine-on-fedora/.

```zsh
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://dl.winehq.org/wine-builds/fedora/33/winehq.repo
sudo dnf -y install winehq-stable
```

After the install:

- `wine --version` to confirm the version
- `winecfg` to configure wine.

Then get winetricks:

```zsh
wget  https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
chmod +x winetricks
sudo mv winetricks /usr/local/bin/
```

### Enable rpmfusion for Handbrake and OBS and VLC
I know a lot of people who favour Linux for _libre_ rather than _gratis_ would not approve of rpmfusion but I'm just being pragmatic: I need Handbrake and VLC.

First, follow instructions at https://rpmfusion.org/Configuration#Command_Line_Setup_using_rpm

Then:

```zsh
sudo dnf install -y handbrake handbrake-gui
sudo dnf install -y obs-studio
sudo dnf install -y vlc
```

### VirtualBox

Follow the instructions at https://rpmfusion.org/Howto/VirtualBox

### GUI configuration
This is really just for me and my preferences.

- Additional themes (e.g. Fitts) can be installed in `~/.themes`
- In file browser, go to Edit &rarr; Preferences and set default view to list view.
- To remove unwanted bookmarks in the file browser, delete the subfolders from the home folder (e.g. "Movies") and then run `xdg-user-dirs-update`.
- Keyboard shortcuts:
	- Run a terminal: <kbd>Win</kbd>+<kbd>T</kbd>
	- Home folder: <kbd>Win</kbd>+<kbd>E</kbd>
	- Move window one workspace up: Disabled
	- Move window one workspace down: Disabled

### Setup folder for custom fonts

```zsh
mkdir /home/b/.local/share/fonts
# then copy whatever fonts you want
fc-cache -fv
```

### Replicate pbcopy

- Complete tasks as per [replicate-pbcopy.md](../tasks/replicate-pbcopy.md)

### Passwordsafe3 support with Pasaffe

Adapted from https://answers.launchpad.net/pasaffe/+question/658239

1. Download latest tar.gz from https://launchpad.net/pasaffe
2. `sudo dnf install -y rpmdevtools python3-pip python3-distutils-extra`
3. `pip3 install unidecode`
4. Navigate to the folder where the latest tar.gz was downloaded
5. `tar xvzf pasaffe-0.57.tar.gz` (replace with equivalent filename)
6. Navigate to extracted folder
7. `python setup.py bdist_rpm`
8. `find dist/*.rpm`
9. `sudo dnf install -y dist/pasaffe-0.57-1.noarch.rpm`
10. If needed, symlink an existing psafe3 database to `~/.local/share/pasaffe/pasaffe.psafe3`

### Connect to a Samba (SMB) network share
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

### Enable Swapfile and Hibernation

1. Complete these steps from http://ctrl.blog/entry/fedora-hibernate.html which are required:

	- The step involving `dracut`
	- The `/etc/systemd/sleep.conf` step, but uncommenting these 3 lines:
		```
		AllowHibernation=yes
		HibernateMode=platform shutdown
		HibernateState=disk
		```
2. Swap file setup as per below based on https://superuser.com/questions/1581885/btrfs-luks-swapfile-how-to-hibernate-on-swapfile/1613639#1613639

	- (Part 1) Create and configure a swap-file the same size as the system memory
		```zsh
		sudo touch /var/swapfile
		sudo chattr +C /var/swapfile
		sudo fallocate --length "$(grep MemTotal /proc/meminfo | awk '{print $2 * 1024}')" /var/swapfile
		sudo chmod 600 /var/swapfile
		sudo mkswap /var/swapfile
		sudo swapon /var/swapfile
		echo '/var/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
		```

	- (Part 1.5) Calculate offset. We need to use the `btrfs_map_physical.c` code, otherwise the calculation won't take into account the idiosyncrasies of the btrfs filesystem.
		```zsh
		mkdir -p ~/00blair/fedorahibernation
		cd ~/00blair/fedorahibernation
		wget https://raw.githubusercontent.com/osandov/osandov-linux/master/scripts/btrfs_map_physical.c
		gcc -O2 -o btrfs_map_physical btrfs_map_physical.c
		sudo ./btrfs_map_physical /var/swapfile | sed -n "2p" | awk "{print \$NF}" >/tmp/swap_physical_offset
		```

	- (Part 2) Set the resume kernel parameters

		```zsh
		SWAP_PHYSICAL_OFFSET=$(cat /tmp/swap_physical_offset)
		SWAP_OFFSET=$(echo "${SWAP_PHYSICAL_OFFSET} / $(getconf PAGESIZE)" | bc)
		SWAP_UUID=$(findmnt -no UUID -T /var/swapfile)
		RESUME_ARGS="resume=UUID=${SWAP_UUID} resume_offset=${SWAP_OFFSET}"
		echo "${RESUME_ARGS}"
		```

	- (Part 2.5) If the output looks correct, set the kernel parameters with grubby:
		```zsh
		sudo grubby --update-kernel=ALL --args="${RESUME_ARGS}"
		```

	- (Part 3) Disable systemd swap space check.
		```zsh
		sudo mkdir -p /etc/systemd/system/systemd-logind.service.d/
		cat <<-EOF | sudo tee /etc/systemd/system/systemd-logind.service.d/override.conf
		[Service]
		Environment=SYSTEMD_BYPASS_HIBERNATION_MEMORY_CHECK=1
		EOF
		sudo mkdir -p /etc/systemd/system/systemd-hibernate.service.d/
		cat <<-EOF | sudo tee /etc/systemd/system/systemd-hibernate.service.d/override.conf
		[Service]
		Environment=SYSTEMD_BYPASS_HIBERNATION_MEMORY_CHECK=1
		EOF
		```

	- (Part 4)  SELinux: allow systemd to access the swap-file.
		```zsh
		cd "$(mktemp -dt)"
		cat <<-EOF | tee systemd_swapfile.te
		module systemd_swapfile 1.0;

		require {
			type init_t;
			type swapfile_t;
			class file { open getattr read ioctl lock };
		}

		allow init_t swapfile_t:file { open getattr read ioctl lock };
		EOF
		checkmodule -M -m -o systemd_swapfile.mod systemd_swapfile.te
		semodule_package -o systemd_swapfile.pp -m systemd_swapfile.mod
		sudo semodule -i systemd_swapfile.pp
		cd -
		```

	- (Part 5) Reboot and then try to hibernate using `systemctl hibernate`


### Shellstarterkit equivalent
TODO: send this to https://github.com/blairw/shellstarterkit/

- Install the Cascadia font (`sudo dnf install -y cascadia-fonts-all`) set it in VSC (search for "terminal font" in the preferences)

- Then:

```zsh
git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
curl https://raw.githubusercontent.com/blairw/shellstarterkit/master/resources/dot-p10k.zsh -o ~/.p10k.zsh
code ~/.zshrc
```

In `~/.zshrc`:
- `ZSH_THEME="powerlevel10k/powerlevel10k"`
- `plugins=(git virtualenv)`
- End of file: `[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh`


### GNOME prevent sleep-if-no-one-logs-in

gnome prevent sleep if no one logs in - adapted from https://askubuntu.com/a/1132068

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

## Ongoing administrative tasks

### Usual updates

```zsh
noglob sudo dnf update -y --exclude=kernel*
```

### If you want to try a new version of the kernel

- Ensure that all required packages, including `kernel-headers`, are available for that new version
- Ensure that the new version still works with VirtualBox

### If you ever need to install a specific version of the Linux kernel

As per https://unix.stackexchange.com/questions/475128/is-there-a-quick-way-to-install-earlier-fedora-kernel-packages and https://fedoramagazine.org/install-kernel-koji/

1. Find the specific version of `kernel` at https://koji.fedoraproject.org/koji/packageinfo?packageID=8 (if needed, find the corresponding version of `kernel-headers` at https://koji.fedoraproject.org/koji/packageinfo?packageID=27325)

2. Download all required rpms (to clarify which ones you need, run `sudo dnf list installed | grep kernel`, and anything listed there that is also listed on the koji page, make sure you get from the koji page - but it is ok if some things listed by the dnf output are not on the koji page, they are provided elsewhere - you may wish to find them if necessary)

3. In Terminal, navigate to the rpms and run `sudo dnf install ./kernel*`

4. To remove old kernels, see https://fedingo.com/how-to-remove-unused-kernels-in-rhel-fedora-centos/ - but basically:

```zsh
# Latest 2 kernels only
sudo dnf remove $(dnf repoquery --installonly --latest-limit=-2 -q) 

# Latest 1 kernel only
sudo dnf remove $(dnf repoquery --installonly --latest-limit=-1 -q)
```
