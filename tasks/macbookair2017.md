# Tasks specific to MacBook Air 2017

## Technical specifications (FYI only)

- MacBookAir7,2
- Intel i5-5350U (4) @ 2.9GHz
- Intel HD Graphics 6000

## Dual-booting with macOS

Removing macOS is not recommended for reasons outlined at: See https://www.reddit.com/r/linux4noobs/comments/18kde7q/putting_linux_on_your_mac_watch_out/

## Part 1 - Installation of Ubuntu 22.04 LTS

Unfortunately, the Ubuntu 24.04 LTS installer can only install with LVM and LUKS encryption if you erase the entire disk. This limitation did not apply to the previous LTS release (22.04). So, install that release, then perform an upgrade to Ubuntu 24.04 LTS.

## Part 2 - After installation of Ubuntu 24.04 LTS

### Setup of wifi

```bash
sudo apt install broadcom-sta-common broadcom-sta-dkms
```

As per https://www.lorenzobettini.it/2025/04/installing-ubuntu-24-10-on-an-old-macbook-air-2016/

### Interim Cleanup

```bash
sudo apt install git
sudo apt autoremove
```

### Setup of camera

```bash
cd ~/00blair/gitrepos-collections

# FIRMWARE
sudo apt install xz-utils curl cpio make
git clone https://github.com/patjak/facetimehd-firmware.git
cd facetimehd-firmware
make
sudo make install

cd ..

# DRIVER - NOTE IT IS CHECKINSTALL NOT MAKE INSTALL!
sudo apt install linux-headers-generic git kmod libssl-dev checkinstall
git clone https://github.com/patjak/facetimehd
cd facetimehd
make
sudo checkinstall
```

You will be told that it has produced a file along the lines of `facetimehd_20260408-1_amd64.deb`. Go and install it with `sudo dpkg -i`

```
sudo depmod
sudo modprobe facetimehd
```

As per https://github.com/patjak/facetimehd/wiki/Installation#get-started-on-ubuntu

WARNING: Do NOT attempt DKMS installation (https://github.com/patjak/facetimehd/wiki/Installation#setting-up-dkms-auto-compile-on-kernel-update-1) as `dkms mkdsc` and `dkms mkdeb` are no longer available in new versions of dkms.


## Part 3 - Ergonomics

### Colour profile for built-in display

Get the ICC file as per https://gist.github.com/amercier/e6d118fc73ecf2efb8ca183cbae20c48

Install it by going to **Settings &rarr; Colour &rarr; Laptop Screen** and then clicking the **Add profile** button.

### Disable super (command) key for overview

This is necessary in order to allow for keyboard shortcuts like Command+C, Command+V, etc.

```bash
gsettings get org.gnome.mutter overlay-key     # 'Super_L'
gsettings set org.gnome.mutter overlay-key ''
```

### Remapping keyboard shortcuts using keyd

Follow instructions for building from source: https://github.com/rvaiya/keyd?tab=readme-ov-file#from-source

Then:

```bash
sudo mkdir -p /etc/keyd
sudo nano /etc/keyd/default.conf
```

See [mac-keyd-config.conf](../mac-keyd-config.conf) for what to put in this file.

- To then **reload**, run `sudo keyd reload`
- To **check for issues**, run `sudo systemctl status keyd`

## Part 4 - Notes on Performance

### Alpaca

You can install Alpaca as a Flatpak as per [install-flatpaks.md](install-flatpaks.md).

1B models run decently well on the MacBook Air 2017 (e.g., Llama3.2 1B)

