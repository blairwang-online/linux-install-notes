# Fedora Silverblue Linux setup notes by Blair

## Rationale

### "Immutable" approach to underlying OS

Fedora Silverblue is the GNOME variant of "Fedora Atomic Desktops" (see also Fedora Kinoite for KDE). These take a different approach to package management sometimes referred to as "immutable". Essentially there is an attempt to separate the core of the operating system (kernel and absolutely essential packages) from the user-facing software.

- Most of the core operating systems files are locked down, mounted in a read-only fashion on boot. As per the [documentation](https://docs.fedoraproject.org/en-US/fedora-silverblue/technical-information/#filesystem-layout), only `/etc/` and `/var/` are writeable, so home folders are stored under `/var/home/` instead of `/home/` (`/home/` is set up as a symlink to `/var/home/`).

- The preferred way to install graphical software is using Flatpak, especially software from Flathub.

- The preferred way to install command-line software is using Toolbox created using the regular settings (i.e., the Toolbox container runs Fedora, the same version number as the host, but the non-Silverblue version). You can then go inside the Toolbox and use `dnf` as normal.

- If there is some specific software which cannot be installed using Flatpak or Toolbox but is easy to install in other Linux distros, you can use Distrobox. For example you can set up Debian in a Distrobox and use it to install software for which there are DEB repositories (e.g., Signal, Spotify, VLC, Zoom).

- If you want to add extra packages available at the operating system level (RPM), you can use `rpm-ostree` in a similar way to `dnf` (2015-present) and `yum` (1999-2015), but you're supposed to do this as little as possible and try to use the other methods first (Flatpak, Toolbox, Distrobox). Packages installed using `rpm-ostree` are "layered packages" and considered to be deviations from the clean image. You can see these with `rpm-ostree status`, for example:

    ```
    > rpm-ostree status  
    State: idle
    Deployments:
    fedora:fedora/43/x86_64/silverblue
                    Version: 43.20251209.0 (2025-12-09T00:28:37Z)
                BaseCommit: (REDACTED)
                GPGSignature: Valid signature by (REDACTED)
                        Diff: 27 upgraded
            LayeredPackages: cockpit-files cockpit-networkmanager cockpit-ostree cockpit-podman cockpit-selinux cockpit-storaged cockpit-system cockpit-ws containerd.io ddcutil distrobox docker-buildx-plugin docker-ce docker-ce-cli docker-compose-plugin git nextcloud-client nextcloud-client-nautilus screen zsh
    ```

- This hierarchy of Flatpak > Toolbox > Distrobox > DNF/APT could also be done on enterprise Linux (e.g., RHEL, AlmaLinux, Rocky), CentOS Stream, or even regular Fedora, or even non-RPM distros like Debian and Ubuntu. Basically you are refactoring your setup to offload as much away from the regular package manager and into the isolated environments inside Flatpak / Toolbox / Distrobox.

- But, the main advantage of Silverblue is that you can be 100% sure that the underlying OS is read-only (immutable) and so it can be hot-swapped with an older version of the same underlying OS. Silverblue keeps snapshots of older configurations using rpm-ostree with a similar kind of logic to Git. `rpm-ostree rollback` immediately (without confirming!) goes back to an older state (older versions of packages recently upgraded).

Bazzite takes a similar approach since it is based on Fedora Atomic Desktops. See https://docs.bazzite.gg/General/Fedora_Atomic_Comparison/

### Advantages and disadvantages

Advantages:

- Isolating user-facing software into Flatpaks/Toolbox/Distrobox introduces a bit of stability and longevity. In theory you should be able to upgrade to the latest version of Fedora by hot-swapping out the underlying OS, without any interference to your user data and your software in Flatpaks/Toolbox/Distrobox. Thus the short support window for Fedora releases (see https://endoflife.date/fedora) is less of a worry.

- You can fine-tune Flatpak apps' permissions using Flatseal which is fantastic. For example you can prevent a particular app from accessing the network at all (very useful 'kill switch' for offline AI using Alpaca)

- Therefore I feel a bit safer about using Fedora on my main machine, and thus benefit from the newer software in Fedora (newer kernel, newer GNOME version) - which is not available to me when using a traditional Linux distro with an "LTS" kind of approach.

- Using Toolbox/Distrobox to just install software from regular Fedora and Debian repos is quite handy. It means not relying on unverified Flathub listings, COPRs, etc. This might actually be even more of an advantage on a enterprise Linux release cycle (e.g., AlmaLinux, RHEL) to get around EPEL.

Disadvantages:

- Storage and compute overheads of running software in Flatpaks/Toolbox/Distrobox. The latter two (Toolbox and Distrobox) are essentially wrappers around [Podman](https://podman.io/). It's not quite as much overhead as full-on virtualisation but it's more than nothing.

- Isolating user-facing software doesn't magically eradicate regressions in the host OS itself. For example you might encounter a bug in a newer version of GNOME, or little hardware issues in a newer version of the Linux kernel. So far I've only experienced the tiniest of issues (e.g., trackpad sometimes locking up, only fixed after I log out and log back in). But in principle these could still happen.

- On the rare occasion that you need to install packages into the host OS using `rpm-ostree`, you need to reboot after each of these packages.

- Limitations of Flatpaks:
    - **VSCodium:** Integrated terminal in is very limited (e.g., doesn't have your usual ZSH setup and profile)
    - **KeepassXC:** Missing drop-shadow for main window.

- Limitations of Distrobox:
    - Programs cannot be set as handlers (e.g., VLC).

- You need to be disciplined and vigilant when using the terminal. You need to maintain awareness about "where" you are, e.g., if you are in the underlying OS, or inside the Toolbox, or inside the Debian distrobox, or inside a Python virtual environment inside vs. outside the OS, etc.

## Setup Fedora Silverblue

### Installation

Fedora silverblue specific:

- During setup, do NOT enable third party repos, network/wifi, or NTP. Do NOT enable location services.


### zsh

Go through https://github.com/blairwang-online/linux-install-notes/blob/main/tasks/setup-preferred-shell.md as normal. 

Don't try to do some hybrid thing with Bash in the host and ZSH in the toolbox. Due to issue as follows: https://github.com/containers/toolbox/issues/908

### Disable automatic updates

Before starting, check that `rpm-ostree status` indicates that the state is idle.

sudo nano /etc/rpm-ostreed.conf

Uncomment: AutomaticUpdatePolicy=none

Now reboot and consider this done


### Network shares

For mounts, do not attempt to use the fstab method. fstab is writeable but you cannot chmod /bin/mount so that method doesnt work. Just run on commandline as sudo:

```
sudo mount -t cifs -o rw,credentials=/home/blair/TODO-PUT-HERE-credentials.txt,nounix,dir_mode=0777,file_mode=0777 //your-server/your-share /media/your-share # mount your-share
```

### GNOME Settings

- Power Saving --> Automatic Suspend --> DISABLE when plugged in

### GNOME Extensions

Already installed, just run it and setup your bits.

### Tweaks

gsettings set org.gnome.desktop.wm.preferences button-layout ":minimize,maximize,close"
gsettings set org.gnome.desktop.wm.preferences resize-with-right-button true
gsettings set org.gnome.desktop.wm.preferences mouse-button-modifier '<Alt>'

gsettings get org.gnome.desktop.interface font-hinting
gsettings set org.gnome.desktop.interface font-hinting 'none'


## RPM packages with rpm-ostree (Layered Packages)

1. Try rpm-ostree status, see if busy or idle. If busy then wait until idle.

2. Test if `screen` is installed. If not then rpm-ostree install screen
    - Just that one package as a test case. 'Screen' is chosen because it works poorly inside toolbox.
    - Should end with "Changes queued for next boot. Run "systemctl reboot" to start a reboot"
    
3. Other things for dnf directly onto system

- `ddcutil`
    - To facilitate gnome extension for changing screen brightness
- `nextcloud-client nextcloud-client-nautilus`
    - Because flathub version isnt verified
- `git zsh`
    - For appropriate zsh config


## Flatpaks

Go to Software. It will ask about Third Party Software Repos, click IGNORE. Then --> Hamburger menu (top-right corner) --> Software repositories. Or `Ctrl+.`
    - Then enable Flathub (not Fedora Flatpaks Testing!!)
        - If it's not there: flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
        - Then try to enable it again...

If that's all good then proceed with the full set:

```
flatpak install flathub cz.ondrejkolin.Barcoder
flatpak install flathub io.github.alainm23.planify
flatpak install flathub com.brave.Browser
flatpak install flathub com.github.marhkb.Pods
flatpak install flathub com.github.tchx84.Flatseal
flatpak install flathub com.obsproject.Studio
flatpak install flathub com.vscodium.codium
flatpak install flathub io.github.flattool.Warehouse
flatpak install flathub io.gitlab.librewolf-community
flatpak install flathub io.github.cboxdoerfer.FSearch
flatpak install flathub io.github.quodlibet.QuodLibet
flatpak install flathub net.codelogistics.webapps
flatpak install flathub org.cryptomator.Cryptomator
flatpak install flathub net.nokyan.Resources
flatpak install flathub org.ferdium.Ferdium
flatpak install flathub org.gimp.GIMP
flatpak install flathub org.gnome.World.Secrets
flatpak install flathub org.kde.krita
flatpak install flathub org.keepassxc.KeePassXC
flatpak install flathub org.libreoffice.LibreOffice
flatpak install flathub org.mozilla.Thunderbird
flatpak install flathub org.scratchmark.Scratchmark
flatpak install flathub org.pwsafe.pwsafe
flatpak install flathub page.codeberg.libre_menu_editor.LibreMenuEditor
flatpak install flathub page.tesk.Refine
```

Specific to Alpaca:

```
flatpak install flathub com.jeffser.Alpaca
flatpak install com.jeffser.Alpaca.Plugins.Ollama
```

Note: the following were considered but appeared as "Unverified" on flathub:

- https://flathub.org/en/apps/com.nextcloud.desktopclient.nextcloud


## Toolbox

Use Toolbox to install and run command line programs that are available in regular Fedora but not Fedora Silverblue.

```
toolbox create
toolbox enter
sudo dnf install -y aria2 btop fastfetch glow go nodejs-npm zsh
```

- DO NOT install screen in here. You can launch a new terminal (bash on host system), then launch a new screen, then run toolbox. Then, from that toolbox within a screen, you can do Ctrl+A Ctrl+D and it will get out of that screen as per normal.

- The correct way to run btop is `toolbox run sudo btop`

## Distrobox

Similar concept to toolbox, but allows you to use a different distro. Handy for debian - more software is available as .deb as you know - but do not use this where toolbox suffices.

```
sudo rpm-ostree install distrobox
```

After reboot:

```
distrobox create -i debian:13
# "Trying to pull docker.io/library/debian:13..."
# New tab:
distrobox enter debian-13
sudo apt install fastfetch git
sudo apt install vlc
sudo apt install fonts-noto fonts-noto-color-emoji  # for emoji and non-latin support within apps like Spotify
```


Then inside Debian distrobox:

- https://signal.org/download/#

In the main system: go to ~/.local/share/applications and create signal-desktop.desktop

```                                                  
[Desktop Entry]
Type=Application
Icon=signal-desktop
Name=Signal (in Toolbox)
Name[en]=Signal (in Distrobox)
Exec=distrobox enter debian-13 -- signal-desktop
PrefersNonDefaultGPU=false
Hidden=false
NoDisplay=false
StartupNotify=false
Terminal=false
StartupWMClass=signal
```

- https://www.spotify.com/us/download/linux/
    - Use deb method
    - Then use QR code scan to get in

```                                                  
[Desktop Entry]
Type=Application
Icon=spotify
Name=Spotify (in Distrobox)
Exec=distrobox enter debian-13 -- spotify
PrefersNonDefaultGPU=false
Hidden=false
NoDisplay=false
StartupNotify=false
Terminal=false
StartupWMClass=Spotify
```

- https://zoom.us/download?os=linux
    - Using browser, download deb to downloads folder
    - Then install using sudo dpkg -i ./<filename>

To figure out `StartupWMClass`, do Alt+F2 then `lg` then Windows

Also, VLC:

```                                                  
[Desktop Entry]
Type=Application
Icon=vlc
Name=VLC (in Distrobox)
Exec=distrobox enter debian-13 -- vlc
PrefersNonDefaultGPU=false
Hidden=false
NoDisplay=false
StartupNotify=false
Terminal=false
StartupWMClass=vlc
```


## Docker

Adapted from:

- https://docs.docker.com/engine/install/fedora/
- https://discussion.fedoraproject.org/t/installing-docker-on-silverblue/119610/5

```zsh
curl -O https://download.docker.com/linux/fedora/docker-ce.repo
sudo install -o 0 -g 0 -m644 docker-ce.repo /etc/yum.repos.d/docker-ce.repo
rpm-ostree install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

After reboot:

```zsh
sudo systemctl enable --now docker
```

## Ollama (inside Docker)

See https://dev.to/ajeetraina/how-to-setup-open-webui-with-ollama-and-docker-desktop-24f0

(Basically you just need to clone the open-webui repo, then use the docker-compose.yaml provided. It will set up both ollama and open-webui in the same config.)

Don't forget to go to Cockpit to open port 3000

Then go to http://localhost:3000/ and set up your login detials

Then go to and manage Ollama.

http://localhost:3000/admin/settings/connections

## Cockpit

rpm-ostree install cockpit-files cockpit-networkmanager cockpit-ostree cockpit-podman cockpit-selinux cockpit-storaged cockpit-system cockpit-ws

Then:

sudo systemctl enable --now cockpit.socket


## To disable Airplay (e.g., to avoid sudden sounds on Home speaker)

mkdir -p ~/.config/pipewire/pipewire.conf.d/
nano ~/.config/pipewire/pipewire.conf.d/disable-raop.conf

Edit it to have:

```
context.properties = {
    "module.raop" = false
}
```

systemctl --user status pipewire
systemctl --user restart pipewire

## HTTPD (apache)

sudo systemctl status httpd
sudo systemctl enable httpd
sudo systemctl enable httpd

## Misc

Gnome settings. Search. Disable all search results except Settings

Gnome extensions - https://extensions.gnome.org/extension/3193/blur-my-shell/

https://extensions.gnome.org/extension/1465/desktop-icons/

To run VSC at the command line, opening the current folder:

```zsh
flatpak run com.vscodium.codium $(pwd) &
```

To use LibreOffice to convert DOC/DOCX files to PDF (equivalent to using `soffice` in this way)

```zsh
flatpak run org.libreoffice.LibreOffice --convert-to pdf:writer_pdf_Export *.doc*
```

