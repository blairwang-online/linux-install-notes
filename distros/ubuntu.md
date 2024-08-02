# Ubuntu setup notes by Blair

## Rationale

These notes based on **Ubuntu 24.04 LTS**.

## Install Ubuntu by USB

- Complete tasks as per [install-to-bare-metal.md](../tasks/install-to-bare-metal.md)
- Connect to Wifi so that the installer can update
- Select **Interactive installation**
- Select **Default selection**
- Install recommended proprietary software>
	- YES to both
- How do you want to install Ubuntu?
	- Unless you know you want otherwise (e.g., Dual Boot): Select "Erase disk and install Ubuntu"
	- Click on **Advanced Features**
	- Select **Use LVM and encryption**

## On First Boot

### First boot splash screen

- Skip "Ubuntu Pro" unless you use it
- No, don't share system data - unless you'd like to (it's nice how they show what is being transmitted)

### Initial administrative tasks

- `sudo apt update && sudo apt upgrade`
- Complete tasks as per [setup-preferred-shell.md](../tasks/setup-preferred-shell.md)
- Complete tasks as per [gnome-configurations.md](../tasks/gnome-configurations.md)

## Software Installs

### Server and workstation

```bash
sudo apt-get install -y aria2 bat btop git-lfs lm-sensors jq libreoffice lolcat nodejs npm screen whois
```

### Workstation only

```bash
sudo apt-get install -y audacity cheese dconf-editor ddcutil geany geany-plugins gcolor3 gimp gitg gnome-builder gnome-calendar gnome-contacts gnome-maps gnome-sushi gnome-tweaks gnome-weather nautilus-dropbox nautilus-nextcloud obs-studio pasaffe pavucontrol python3.12-venv synaptic ttf-mscorefonts-installer syncthing tree vlc

# Snaps where the apt/deb version is problematic or unavailable
sudo snap install 0ad chromium emote
```

Fonts:

```bash
sudo apt-get install fonts-clear-sans fonts-comic-neue fonts-courier-prime fonts-inconsolata fonts-inter fonts-open-sans fonts-ricty-diminished fonts-roboto
```

Manual installations:

- **Brave:** From "App Center" (snaps) (yes it is available as `.deb` as well, but let's keep it simple given how infrequently we use this)
- **fastfetch:** https://github.com/fastfetch-cli/fastfetch
- **Ferdium (e.g., for WhatsApp):** Unfortunately the version from "App Center" (snaps) doesn't work; instead go to https://github.com/ferdium/ferdium-app/releases, download the `.deb` file, and install using `sudo dpkg -i`
- **FSearch:** https://github.com/cboxdoerfer/fsearch?tab=readme-ov-file#download and follow instructions for Ubuntu PPA Stable
- **Signal:** https://signal.org/download/linux/
- **Spotify:** https://www.spotify.com/de-en/download/linux/ (use the "Snap" method; the `.deb` method doesn't allow login unfortunately)
- **Syncthing:** https://www.atlantic.net/dedicated-server-hosting/how-to-install-syncthing-on-ubuntu-22-04/
	- Don't forget to configure the web UI (0.0.0.0 for GUI IP address, set HTTPS, set username and password, disable all the NAT traversal and discovery things)
- **Vivaldi:** https://vivaldi.com/download/
- **VSCodium:** https://vscodium.com/#install-on-debian-ubuntu-deb-package
- **Zed:** https://zed.dev/download


## Software Configurations

### Enable Cockpit

Follow instructions at https://cockpit-project.org/running#ubuntu

Then:

```bash
sudo apt-get install cockpit-machines
```

(As per https://askubuntu.com/questions/1403072/what-package-manager-can-i-use-to-install-cockpit-machine-on-ubuntu-server)

Then:

- Overview → View metrics and history → Install cockpit-pcp
- Then logout, log back in
- Overview → View metrics and history → Metrics settings and enable Collect metrics but maybe not Export to network (!).
- Applications → Machines → Install

### Enable SSH

- Confirm not yet running: `sudo systemctl status ssh`
- `sudo apt-get install -y openssh-server`
- That's it; no further action required
- Confirm now running: `sudo systemctl status ssh`

### Replicate pbcopy and pbpaste

- Complete tasks as per [replicate-pbcopy.md](../tasks/replicate-pbcopy.md)

### OBS config

On first run:

- Optimise for recording (unless you know you will be streaming)

Afterwards:

- Remember to set output canvas back up to 1920x1080

### Samba (SMB) - client-side configuration (if this machine will connect to shares)

```
sudo apt-get install cifs-utils
sudo mkdir /media/myshare1
sudo mkdir /media/myshare2

sudo mount -t cifs -o rw,username=blair,nounix,dir_mode=0777,file_mode=0777 //server1/myshare1 /media/myshare1 # mount myshare1
sudo mount -t cifs -o rw,username=blair,nounix,dir_mode=0777,file_mode=0777 //server1/myshare2 /media/myshare1 # mount myshare2

sudo umount /media/myshare1
sudo umount /media/myshare2
```

Some instructions use `/mnt/` instead of `/media/`. The main difference as far as I am concerned is that using `/media/` makes it appear as a removable drive in Nautilus file manager. For more information please refer to https://askubuntu.com/questions/22215/why-have-both-mnt-and-media


### Git / GitHub credentials saver

- Complete tasks as per [git-github-credentials-saver.md](../tasks/git-github-credentials-saver.md)

### Mozilla PPA for Firefox ESR and Thunderbird

- Go to snap store and remove Firefox
- Add https://launchpad.net/~mozillateam/+archive/ubuntu/ppa
- Install Firefox ESR https://ubuntuhandbook.org/index.php/2023/09/install-firefox-esr-115-ppa/
- Install Thunderbird: https://ubuntuhandbook.org/index.php/2024/03/install-thunderbird-deb-ubuntu-2404/

### Firefox Configuration

Recommended extensions:

1. https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/
2. https://ublockorigin.com/

Recommended about:config settings:

- `pdfjs.defaultZoomValue` = `page-fit` (as per https://support.mozilla.org/bm/questions/1106654)
- `pdfjs.spreadModeOnLoad` = `1` (as per https://stackoverflow.com/questions/65401710/pdf-js-default-spread-setting)

### GNOME Settings

- Sound -> Alert Sound -> None

### GNOME Tweaks

- Fonts:
	- Hinting = None
	- Antialiasing = Standard (grayscale)
- Mouse:
	- Middle Click Paste = OFF
	- Pointing Stick Acceleration = OFF
- Keyboard:
	- Additional Layout Options --> Caps Lock behavior --> Make Caps Lock an additional Ctrl
- Windows:
	- Attach Modal Dialogs = OFF

### GNOME Extensions

```bash
sudo apt-get install gnome-shell-extension-manager

# As per https://github.com/elvetemedve/gnome-shell-extension-system-monitor/wiki/Installation#dependencies
sudo apt install gir1.2-gtop-2.0 libgtop2-dev
```

You should reboot after this point (to ensure libgtop installs properly)

Recommended extensions which use libgtop:

- "System Monitor" by fmuellner

I found that this one causes the GNOME session to crash:

- "System Monitor" by elvetemedve

Other recommended extensions:

- "Wiggle" by mechtifs


### GNOME Builder

- Some configurations (e.g., right margin shaded area) are on a per-language basis
- Go to Preferences (Ctrl+,) then **Programming Languages**, select the language you want to configure (e.g., Markdown).

### VSCodium

- Complete tasks as per [configure-vsc.md](../tasks/configure-vsc.md)

## Hardware Support

### Logitech MX Anywhere mouse

- Install Solaar: `sudo apt-get install solaar`
- Run Solaar: `sudo solaar` (if you do not run as root, it may not be able to detect your mouse)
- Remap the middle key as per the example at https://github.com/pwr-Solaar/Solaar/issues/2246


### Apple Keyboard

Fn mode:


```bash
echo 2 | sudo tee /sys/module/hid_apple/parameters/fnmode
```

As per https://help.ubuntu.com/community/AppleKeyboard#Change_Function_Key_behavior


## Optional Installations and Configurations

### Flatpak (needed for Planify)

https://flatpak.org/setup/Ubuntu

Planify: `io.github.alainm23.planify` (similar to Things)


### Waterfox

As at time of writing, only tar.gz available for Linux.

1. Go to https://www.waterfox.net/download/ and download latest release for 🐧 **Linux**.
2. Extract it to, e.g., `~/00blair/bin`
3. For version-agnostic symlink: `ln -s ~/00blair/bin/waterfox-G6.0.17/waterfox ~/00blair/bin/waterfox`
4. Set up a launcher shortcut:

```bash
sudo nano /usr/share/applications/waterfox.desktop
```

Copy the contents of [waterfox.desktop](../config-files/waterfox.desktop)



## Regular administration and maintenance

### Update Ubuntu App Store (snap store)

```bash
sudo snap refresh snap-store
```

### Fix regular "not responding" prompts

```bash
~
❯ gsettings get org.gnome.mutter check-alive-timeout
uint32 5000

~
❯ gsettings set org.gnome.mutter check-alive-timeout 60000
```

From https://askubuntu.com/a/1215995

