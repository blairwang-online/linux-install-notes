# Forgejo in a Podman container

Self-hosted Git forge on AlmaLinux 10, run as a rootful Podman Quadlet so systemd starts it at boot. Adapted from the Podman Quadlet example in https://forgejo.org/docs/latest/admin/installation-docker/ (tested by Forgejo on AlmaLinux 9.4 with Podman 5.2). Not tested end-to-end here yet.

## Which version

Use the LTS tag. Forgejo 15 is the current LTS, supported until 15 July 2027, and the `15` tag always points at the latest 15.x release. The next LTS is expected to be 19 (April 2027). Moving between major versions (for example 15 to 19) is a manual step: read the release notes and back up first.

Forgejo 15's container images trust reverse-proxy headers from anywhere by default (`REVERSE_PROXY_TRUSTED_PROXIES = *`; changed in 16, but not in the 15 LTS). That only matters if reverse-proxy authentication is turned on, but the setting below narrows it anyway.

## Install

Replace `myserver` with the machine's hostname. Clone URLs are built from these settings, so pick the hostname before you start.

```zsh
sudo dnf install -y podman
sudo mkdir -p /srv/forgejo
```

Create `/etc/containers/systemd/forgejo.network`:

```ini
[Network]
NetworkName=forgejo
```

Create `/etc/containers/systemd/forgejo.container`:

```ini
[Unit]
Description=Forgejo

[Container]
ContainerName=forgejo
Image=codeberg.org/forgejo/forgejo:15
AutoUpdate=registry
Network=forgejo.network
PublishPort=3000:3000
PublishPort=222:22
Volume=/srv/forgejo:/data:Z
Environment=USER_UID=1000
Environment=USER_GID=1000
Environment=FORGEJO__server__DOMAIN=myserver
Environment=FORGEJO__server__ROOT_URL=http://myserver:3000/
Environment=FORGEJO__server__SSH_PORT=222
Environment=FORGEJO__security__REVERSE_PROXY_TRUSTED_PROXIES=127.0.0.0/8,::1/128

[Service]
Restart=always

[Install]
WantedBy=default.target
```

Notes on these settings:

- Port 222 is for Git over SSH, leaving 22 for the host's own SSH. `SSH_PORT=222` makes Forgejo show `:222` in its clone URLs; inside the container, SSH still listens on 22.
- `/srv/forgejo` holds everything (repositories, the SQLite database, `gitea/conf/app.ini`). `:Z` gives it the SELinux label the container needs. Don't use `:Z` on system folders or anything under `/home`.
- `USER_UID=1000` means files in `/srv/forgejo` are owned by host UID 1000, which is normally the first user created at install.
- `FORGEJO__section__KEY` variables are written into `app.ini` each time the container starts. They can add or change settings but not remove them; edit `app.ini` for that.

Then start it and open the firewall:

```zsh
sudo systemctl daemon-reload
sudo systemctl start forgejo
sudo firewall-cmd --permanent --add-port=3000/tcp --add-port=222/tcp
sudo firewall-cmd --reload
```

Don't run `systemctl enable forgejo`. Quadlet generates the service, and the `[Install]` section is what starts it at boot; `enable` fails on a generated unit.

Go to `http://myserver:3000` to finish setup. SQLite (the default) is fine for a small personal server. Create the admin account there.

Test SSH once your key is added in Forgejo:

```zsh
ssh -F /dev/null -p 222 git@myserver
```

## Updates

`AutoUpdate=registry` lets Podman pull newer 15.x images. Turn on the timer that does it:

```zsh
sudo systemctl enable --now podman-auto-update.timer
```

To update by hand: `sudo podman auto-update`.

## Backups

Everything is in `/srv/forgejo`. For a consistent copy, stop the service first:

```zsh
sudo systemctl stop forgejo
# copy or snapshot /srv/forgejo
sudo systemctl start forgejo
```

## Troubleshooting

- Logs: `sudo journalctl -u forgejo` or `sudo podman logs forgejo`.
- If the container can't write to `/srv/forgejo`, check for SELinux denials: `sudo ausearch -m avc -ts recent`.
- Cockpit's _Podman containers_ page (`cockpit-podman`) shows the container and its logs.
