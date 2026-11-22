# Ubuntu setup notes by Blair

## Scope

Tested with **Ubuntu 24.04 LTS** and **Ubuntu 26.04 LTS**.

## Install Ubuntu by USB

- Complete tasks as per [install-to-bare-metal.md](../tasks/install-to-bare-metal.md)
- Connect to Wifi so that the installer can update
	- Skip this step if you want to exclusively use a specific apt server like `aarnet`.
- Select **Interactive installation**
- Select **Default selection**
- Install recommended proprietary software
	- YES to both
- How do you want to install Ubuntu?
	- Unless you know you want otherwise (e.g., Dual Boot): Select "Erase disk and install Ubuntu"
	- On Ubuntu 24.04 LTS:
		- Click on **Advanced Features**
		- Select **Use LVM and encryption**
	- On Ubuntu 26.04 LTS:
		- You will be prompted for if and how you want to set up disk encryption.

## On First Boot

### First boot splash screen

- Skip "Ubuntu Pro" unless you use it
- No, don't share system data - unless you'd like to (it's nice how they show what is being transmitted)

### Configure apt server

- Complete the tasks specified at [ubuntu-configure-apt-mirror.md](../tasks/ubuntu-configure-apt-mirror.md)

### Initial administrative tasks

- `sudo apt update && sudo apt upgrade`
- Complete tasks as per [setup-preferred-shell.md](../tasks/setup-preferred-shell.md)
- Complete tasks as per [gnome-configurations.md](../tasks/gnome-configurations.md)

## Software Installs

### Setup Flatpak

See https://flatpak.org/setup/Ubuntu

Then complete tasks as per [install-flatpaks.md](../tasks/install-flatpaks.md)

### Server and workstation

```bash
sudo apt install aria2 bat btop hyfetch lm-sensors jq lolcat nodejs npm pipx screen whois
```

### Workstation only

Note: if gnome-sushi is not working (an issue with Ubuntu 26.04 LTS as at 03 May 2026) - then you can install it from Flathub. See https://flathub.org/en/apps/org.gnome.NautilusPreviewer

```bash
sudo apt install audacity ffmpeg geany geany-plugins gcolor3 gimp gitg gnome-contacts gnome-maps gnome-tweaks gnome-weather meld nautilus-nextcloud showtime synaptic tree vlc wireshark wl-clipboard

# Snaps where the apt/deb version is problematic or unavailable
sudo snap install 0ad
```

Manual installations:

- **Signal:** https://signal.org/download/linux/
- **Syncthing:** https://www.atlantic.net/dedicated-server-hosting/how-to-install-syncthing-on-ubuntu-22-04/
	- Don't forget to configure the web UI (0.0.0.0 for GUI IP address, set HTTPS, set username and password, disable all the NAT traversal and discovery things)

### pandoc and pdflatex

sudo apt install pandoc texlive-full

### Enable support for HEIC images

```bash
sudo apt install libheif-plugin-libde265
```

As advised by https://www.omgubuntu.co.uk/2026/05/fix-heic-images-not-loading-ubuntu-26-04

### LAMP server

First, install the vital components:

```bash
sudo apt update
sudo apt install apache2 libapache2-mod-php php php-bcmath php-curl php-imagick php-intl php-json php-mbstring php-mysql php-xml php-zip

sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2

sudo apt install mariadb-server
sudo systemctl status mysql
sudo mysql_secure_installation
```

(Loosely adapted from https://ubuntu.com/tutorials/install-and-configure-wordpress but uses mariadb instead of mysql.)

Next, install phpmyadmin:

```bash
sudo apt-get install phpmyadmin
```

Configuring phpmyadmin:

- Web server to reconfigure automatically: `apache2`
- Configure database for phpmyadmin with dbconfig-common? `Yes`
- Please provide a password for phpmyadmin to register with the database server: _(generate one yourself, save it in your password manager)_

Configuring PHP:

1. Do the phpinfo thing, find out where php.ini is
2. Open php.ini for editing as root
3. Update `upload_max_filesize = 2M`, maybe `2G`?
4. Update `post_max_size = 8M`, maybe `8G`?


## Software Configurations

### System Settings

Search:

- Turn off most items

### Time sync (NTS)

- Complete tasks as per [configure-nts-time-sync.md](../tasks/configure-nts-time-sync.md) (the steps differ between 24.04 and 26.04)


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

### Samba (SMB)

- Complete tasks as per [setup-samba.md](../tasks/setup-samba.md), client side (if this machine will connect to shares) and/or server side (if this machine will host shares)


### Git / GitHub credentials saver

- Complete tasks as per [git-and-github-configuration.md](../tasks/git-and-github-configuration.md)

### Firefox Configuration

Recommended extensions:

1. https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/
2. https://ublockorigin.com/

Recommended about:config settings:

- `pdfjs.defaultZoomValue` = `page-fit` (as per https://support.mozilla.org/bm/questions/1106654)
- `pdfjs.spreadModeOnLoad` = `1` (as per https://stackoverflow.com/questions/65401710/pdf-js-default-spread-setting)

### Enable webm files to have thumbnails generated for them

```bash
sudo apt update
sudo apt install ffmpegthumbnailer gstreamer1.0-libav gstreamer1.0-plugins-ugly
```

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
```

Then:

- Complete tasks as per [gnome-extensions.md](../tasks/gnome-extensions.md)


### GNOME Builder

- Some configurations (e.g., right margin shaded area) are on a per-language basis
- Go to Preferences (Ctrl+,) then **Programming Languages**, select the language you want to configure (e.g., Markdown).


### GNOME Resize anywhere

By default you can resize anywhere in the window by pressing down the Windows (Super / Command) key and using the middle mouse button.
However, on laptops you may not have access to the middle mouse button.
This can be remedied by changing the setting to use the right mouse (trackpad) button instead.

```
~
❯ gsettings get org.gnome.desktop.wm.preferences resize-with-right-button
false

~
❯ gsettings set org.gnome.desktop.wm.preferences resize-with-right-button true

~
❯ gsettings get org.gnome.desktop.wm.preferences resize-with-right-button     
true
```

As per https://askubuntu.com/a/1239668


### TypeScript

```bash
sudo npm install -g typescript
```

### VSCodium

- Install it via Flatpak (`flatpak install flathub com.vscodium.codium`), as per [install-flatpaks.md](../tasks/install-flatpaks.md)
- Complete tasks as per [configure-vsc.md](../tasks/configure-vsc.md)


### Zotero

- https://www.zotero.org/support/installation points to https://github.com/retorquere/zotero-deb


## Snap removals

Note: if any of these cause errors, then reconsider that line item.

```bash
snap list
snap remove firefox
```


## Artificial Intelligence


### Stable Diffusion

```bash
# Helps with memory I'm told
sudo apt install libtcmalloc-minimal4t64

# Install
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui
python3.11 -m venv .venv
source ./.venv/bin/activate
python -m pip install --upgrade pip wheel
python -m pip install -r requirements.txt

# Run cf. https://github.com/openvinotoolkit/stable-diffusion-webui/wiki/Installation-on-Intel-Silicon
export PYTORCH_TRACING_MODE=TORCHFX && export COMMANDLINE_ARGS="--skip-torch-cuda-test --precision full --no-half" && export python_cmd="python3.11" && ./webui.sh
```

## Hardware Support

### Logitech G203 Lightsync

Simply use Piper:

`sudo apt-get install piper`

Don't forget to click **Apply!**

For a warm white, I find that `#ffffc8` (= RGB 255, 255, 0) is quite nice.


### Hidpi ('retina') screens

Add `--ozone-platform=wayland` to the following:

- `sudo nano /var/lib/snapd/desktop/applications/chromium_chromium.desktop` (use Ctrl+W and search for `Exec=`, could be around 4 instances!)
- `sudo nano /var/lib/snapd/desktop/applications/spotify_spotify.desktop`
- `sudo nano /usr/share/applications/ferdium.desktop`

VSCodium is a Flatpak, so there's no `/usr/share/applications/codium.desktop`. To test, run `flatpak run com.vscodium.codium --ozone-platform=wayland`. To make it stick, copy the Flatpak's launcher into your own folder, where it takes precedence over the system one, then add `--ozone-platform=wayland` right after `com.vscodium.codium` on each `Exec=` line (not tested yet):

```bash
cp /var/lib/flatpak/exports/share/applications/com.vscodium.codium.desktop ~/.local/share/applications/
nano ~/.local/share/applications/com.vscodium.codium.desktop
```

NOTE: It might not work immediately. You might find that you need to launch the application, it is still blurry, then you close it (Alt+F4) and then launch it up again from the launcher, the second time around it should be nice and crispy!

With many thanks to [modzilla99](https://github.com/modzilla99) for the tip over at https://github.com/microsoft/vscode/issues/135492#issuecomment-961762593

Great resource for finding where these desktop launchers live: https://askubuntu.com/a/1144568


### Logitech MX Anywhere mouse

- Install Solaar: `sudo apt-get install solaar`
- Run Solaar: `sudo solaar` (if you do not run as root, it may not be able to detect your mouse)
- Remap the middle key as per the example at https://github.com/pwr-Solaar/Solaar/issues/2246


### Apple Keyboard

The following will give you a setup equivalent to a Windows keyboard with the Ctrl-Fn-Win-Alt layout:

```bash
echo 1 | sudo tee /sys/module/hid_apple/parameters/swap_fn_leftctrl
echo 1 | sudo tee /sys/module/hid_apple/parameters/swap_opt_cmd
```

The following will set Fn lock (since there is no option on the keyboard itself):

```bash
echo 2 | sudo tee /sys/module/hid_apple/parameters/fnmode
```

As per https://help.ubuntu.com/community/AppleKeyboard#Change_Function_Key_behavior

## Cautions

### Cautions about Snaps

- `sudo apt install chromium` will install a Snap

Superseded notes (LibreOffice spellcheck, Mozilla PPA, Waterfox, Ollama and others) are in the [archive](../archive/distros/ubuntu.md).

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

### Fix cups-browsed taking up disproportionately high CPU usage

The `cups-browsed` service provides network printer detection. But using `btop` you may diagnose it to be using the most CPU time of all running programs!

You can verify using `systemctl` that the CPU time is huge (> 1 hour). In comparison some other services' CPU usage is measured in seconds or even just ms.

```
❯ sudo systemctl status cups-browsed 
● cups-browsed.service - Make remote CUPS printers available locally
     Loaded: loaded (/usr/lib/systemd/system/cups-browsed.service; enabled; preset: enabled)
     Active: deactivating (stop-sigterm) since Wed 2024-08-14 01:46:23 IST; 38s ago
   Main PID: 631100 (cups-browsed)
      Tasks: 6 (limit: 76695)
     Memory: 4.9M (peak: 5.5M)
        CPU: 1h 46min 47.776s
     CGroup: /system.slice/cups-browsed.service
             └─631100 /usr/sbin/cups-browsed
```

This is a known issue: https://bugs.launchpad.net/ubuntu/+source/cups-browsed/+bug/2018504

As a provisional workaround, `sudo systemctl disable cups-browsed` (it is enabled by default).

We can re-enable it if we need network printing services, and/or if they fix the bug.

### Not able to launch GDM / GNOME Desktop

This could happen for example if you accidentally installed `fuse` and it removed Ubuntu Desktop.

To fix:

1. Ctrl + Alt + F3 and login by terminal
2. Run `sudo apt install --reinstall ubuntu-desktop`
3. Reboot

Many thanks to https://askubuntu.com/questions/1489129/gdm-service-preventing-boot


