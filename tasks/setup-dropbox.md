# Setup Dropbox

Get this started sooner rather than later, so that it can start fetching your files in the background.

I use Dropbox's headless version because it works across server and workstation environments, across desktop environments, with no dependency on package managers and so on. Don't install Dropbox from GNOME Software.

(On Fedora Workstation I used to install the RPM from https://www.dropbox.com/install-linux instead. I now use the headless version everywhere.)

## Install and link

```bash
# Install
cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -

# Get Dropbox py (the command-line tool)
mkdir -p ~/00blair/dropboxpy
wget -O ~/00blair/dropboxpy/dropbox.py "https://www.dropbox.com/download?dl=packages/dropbox.py"
chmod +x ~/00blair/dropboxpy/dropbox.py

# Run (it prints a link to open in a browser, to link this computer to your account)
~/.dropbox-dist/dropboxd
```

## Setup exclusions

`dropboxd` keeps running in the terminal, so do this in a second terminal once it's linked:

```bash
~/00blair/dropboxpy/dropbox.py exclude add ~/Dropbox/blair-extras2
~/00blair/dropboxpy/dropbox.py exclude add ~/Dropbox/blair-extras3
~/00blair/dropboxpy/dropbox.py exclude add ~/Dropbox/blair-extras4
```

## Firewall for LAN sync

On distros with firewalld (AlmaLinux, Fedora), you will need to configure the firewall to enable LAN sync properly:

```bash
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

On distros with ufw (Ubuntu, Debian), there's nothing to do unless ufw is switched on (`sudo ufw status`). If it is, open the same port that firewalld's `dropbox-lansync` service uses:

```bash
sudo ufw allow 17500 comment 'Dropbox LAN sync'
```
