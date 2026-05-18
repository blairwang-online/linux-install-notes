# Ubuntu - digital forensics workstation

## Scope

Tested with **Ubuntu 24.04 LTS**.

## Getting Ubuntu

Obtain the installer from https://mirror.aarnet.edu.au/pub/ubuntu/releases/24.04/

`ubuntu-24.04-desktop-amd64.iso`

## Immediately after installing Ubuntu

Complete the tasks specified at [ubuntu-configure-apt-mirror.md](../tasks/ubuntu-configure-apt-mirror.md)

## Specific software packages

### Autopsy

```bash
sudo snap install autopsy
```

### Okteta (hex editor)

See https://flatpak.org/setup/Ubuntu

```bash
# Step 1 - install flatpak
sudo apt install flatpak

# Step 2 - add flathub as repo
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Step 3 - install Okteta
flatpak install flathub org.kde.okteta
```

### Volatility version 2

First, you need to set up Python version 2 - complete the tasks specified at [python2-on-ubuntu-2404-2604.md](../tasks/python2-on-ubuntu-2404-2604.md).

Once you have done all of that,

https://github.com/volatilityfoundation/volatility/releases/latest

Once unzipped, you can then run `~/Downloads/volatility_2.6_lin64_standalone/volatility_2.6_lin64_standalone`

For example:

```bash
VOLATILITY=~/Downloads/volatility_2.6_lin64_standalone/volatility_2.6_lin64_standalone
$VOLATILITY -f ~/Desktop/Lab-data/jo-2009-12-11.mdramimage connections > connections.txt
```

### Viking (analysing GPS data)

```bash
sudo apt install viking
```
