# Ubuntu - configure apt mirror

NOTE: This part has been tested with Ubuntu 26.04 LTS and Ubuntu 24.04 LTS only.

Edit the sources file:

```bash
sudo nano /etc/apt/sources.list.d/ubuntu.sources
```

Default:

```
Types: deb
URIs: http://archive.ubuntu.com/ubuntu/
Suites: resolute resolute-updates resolute-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: resolute-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

New (updated URIs):

```
Types: deb
URIs: https://mirror.aarnet.edu.au/pub/ubuntu/archive/
Suites: resolute resolute-updates resolute-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: https://mirror.aarnet.edu.au/pub/ubuntu/archive/
Suites: resolute-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

I'm using the AARNET mirror; update URIs according to your local needs.

