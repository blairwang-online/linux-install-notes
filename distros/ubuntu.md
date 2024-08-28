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
sudo apt-get install -y audacity cheese dconf-editor ddcutil exiftool geany geany-plugins gcolor3 gimp gitg gnome-builder gnome-calendar gnome-contacts gnome-maps gnome-sushi gnome-tweaks gnome-weather meld nautilus-dropbox nautilus-nextcloud obs-studio pasaffe pavucontrol python3.12-venv synaptic ttf-mscorefonts-installer syncthing tree vlc

# Snaps where the apt/deb version is problematic or unavailable
sudo snap install 0ad chromium emote
```

Fonts:

```bash
sudo apt-get install fonts-clear-sans fonts-comic-neue fonts-courier-prime fonts-inconsolata fonts-inter fonts-open-sans fonts-ricty-diminished fonts-roboto

# Do not use because outdated
# sudo apt install fonts-ebgaramond fonts-ebgaramond-extra
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

### NTP setup

```bash
sudo nano /etc/systemd/timesyncd.conf
```

Set these;

```
NTP=0.europe.pool.ntp.org 1.europe.pool.ntp.org 2.europe.pool.ntp.org 3.europe.pool.ntp.org
FallbackNTP=0.pool.ntp.org 1.pool.ntp.org
```

Then:

```bash
sudo systemctl daemon-reload
sudo timedatectl set-ntp off
sudo timedatectl set-ntp on

timedatectl status
```

See https://askubuntu.com/a/1106011

### If you need Docker

Follow instructions at https://docs.docker.com/engine/install/ubuntu/

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


### LibreOffice

1. Close all LibreOffice running programs (Writer, Calc, etc.)
2. `sudo apt install hunspell-en-au hunspell-en-ca hunspell-en-gb`
3. Open LibreOffice Writer, go to **Tools &rarr; Options &rarr; Languages and Locales &rarr; General**. Set **Locale setting** to, e.g., `English (Australia)`.
4. Now go to **Tools &rarr; Options &rarr; Languages and Locales &rarr; Writing Aids**. In the "Available Language Modules" box, click on **Hunspell SpellChecker**, then click **Edit**. Select the relevant language and confirm that **Hunspell SpellChecker** is available and checked (ticked).

### TypeScript

```bash
sudo npm install -g typescript
```

### VSCodium

- Complete tasks as per [configure-vsc.md](../tasks/configure-vsc.md)


### Zotero

- https://www.zotero.org/support/installation points to https://github.com/retorquere/zotero-deb


## Artificial Intelligence

### Ollama

**Installing**

Use the script method at https://ollama.com/download/linux

After installation, check localhost:11434 to see if ollama is running, you can also run `sudo systemctl status ollama` if you're curious. But then you should probably `sudo systemctl disable ollama` and `sudo systemctl stop ollama` so that it only runs when you need it. Unless this is a server.

To install a model, go to https://ollama.com/library and browse for a model you like, e.g., `llama3.1`, then you can run in the terminal:

```bash
ollama pull llama3.1
```

For a nice web UI:

```bash
python3.11 -m pipx install open-webui
open-webui serve
```

**Starting up**

```bash
sudo systemctl start ollama
open-webui serve
```

### Reor

Download the latest AppImage at https://github.com/reorproject/reor/releases/tag/v0.2.18

Try to run as normal but if you get an error like "Rather than run without sandboxing I'm aborting now." then try this:

```bash
./Reor_0.2.18.AppImage --no-sandbox
```

Reor seems to automatically pick up the Ollama models in `/usr/share/ollama/.ollama/models/`.

### aTrain

First you will need to `sudo apt install ffmpeg python3.11 python3.11-venv`

You can then follow the rest of the "Debian" instructions:

```bash
# Go to the folder you want to setup ATrain in
python3.11 -m venv atrain_venv
source atrain_venv/bin/activate

# Check venv in use
which python

# Update pip
python -m pip install --upgrade pip

# Install aTrain
python -m pip install aTrain@git+https://github.com/BANDAS-Center/aTrain.git --extra-index-url https://download.pytorch.org/whl/cu118

# Next, you need aTrain to download all the necessary models for transcription and speaker detection. This has only to be done once when installing it.
aTrain init

# You are now ready to start aTrain. The following command should open a webrowser window with the user interface.
aTrain start
```

(From https://github.com/JuergenFleiss/aTrain/wiki/Linux-Support but adapted for Ubuntu here)


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

<!--

Previously I used OpenRGB. That also works, but Piper is better.

This mouse works very nicely with OpenRGB.

1. Run `sudo apt install libfuse-dev` so that you can run AppImages.
	- WARNING: Do NOT install `fuse`, it will [break your system](https://askubuntu.com/questions/1409496/how-to-safely-install-fuse-on-ubuntu-22-04).
2. Go to https://openrgb.org/releases.html and get the **AppImage**.
3. Run `chmod +x` on the AppImage.
4. Run the AppImage. You may be prompted to install udev rules over at https://openrgb.org/udev - if so then go ahead.

-->

### Hidpi ('retina') screens

Add `--ozone-platform=wayland` to the following:

- `sudo nano /var/lib/snapd/desktop/applications/chromium_chromium.desktop` (use Ctrl+W and search for `Exec=`, could be around 4 instances!)
- `sudo nano /var/lib/snapd/desktop/applications/spotify_spotify.desktop`
- `sudo nano /usr/share/applications/ferdium.desktop`
- `sudo nano /usr/share/applications/codium.desktop`

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


