# Forgejo in a Podman container

Self-hosted Git forge on AlmaLinux 10, run as a rootful Podman Quadlet so systemd starts it at boot. Adapted from the Podman Quadlet example in https://forgejo.org/docs/latest/admin/installation-docker/ (tested by Forgejo on AlmaLinux 9.4 with Podman 5.2). Not tested end-to-end here yet.

## Which version

Use the LTS tag. Forgejo 15 is the current LTS, supported until 15 July 2027, and the `15` tag always points at the latest 15.x release. The next LTS is expected to be 19 (April 2027). Moving between major versions (for example 15 to 19) is a manual step: read the release notes and back up first.

Forgejo 15's container images trust reverse-proxy headers from anywhere by default (`REVERSE_PROXY_TRUSTED_PROXIES = *`; changed in 16, but not in the 15 LTS). That only matters behind a reverse proxy, but the setting below narrows it to localhost anyway. It controls which addresses Forgejo trusts to pass on headers such as `X-Forwarded-For`, not who can connect, so don't widen it to the LAN.

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

Go to `http://myserver:3000` to finish setup. SQLite (the default) is fine for a small personal server. Create the admin account there. Do this straight away: until the setup page has been completed, anyone who can reach port 3000 can complete it.

Test SSH once your key is added in Forgejo:

```zsh
ssh -F /dev/null -p 222 git@myserver
```

## Access from other machines

Nothing in `forgejo.container` limits who can connect. `PublishPort` listens on all of the host's network interfaces and the firewall ports are open, so any machine on the LAN can reach Forgejo at `http://myserver:3000`, or at the machine's IP address. To limit it to this machine instead, use `PublishPort=127.0.0.1:3000:3000`.

`DOMAIN` and `ROOT_URL` don't control access either. They decide the links Forgejo generates: clone URLs, redirects after sign-in, and links in emails and webhooks. Forgejo still answers on any address, but if you browse by an address other than `ROOT_URL`, it shows a warning banner and some links point to the `ROOT_URL` address. So choose the one address you'll normally use:

- **A hostname every network can resolve.** This is the better option for a machine that moves between networks. With mDNS, `myserver.local` works on most LANs without any router changes:

    ```zsh
    sudo dnf install -y avahi
    sudo systemctl enable --now avahi-daemon
    sudo firewall-cmd --permanent --add-service=mdns
    sudo firewall-cmd --reload
    ```

    Then use `myserver.local` in the settings (`DOMAIN=myserver.local`, `ROOT_URL=http://myserver.local:3000/`). Recent Windows, macOS and most Linux desktops resolve `.local` names. A VPN address that doesn't change (WireGuard, Tailscale) works the same way.

- **An IP address**, if it's fixed by a DHCP reservation or a static address:

    ```ini
    Environment=FORGEJO__server__DOMAIN=192.168.1.50
    Environment=FORGEJO__server__ROOT_URL=http://192.168.1.50:3000/
    ```

    On a machine that moves between networks, the address usually differs on each one, so the links will be wrong somewhere.

Either way, leave `SSH_PORT` and `REVERSE_PROXY_TRUSTED_PROXIES` as they are. After changing the settings, restart and check that they reached `app.ini`:

```zsh
sudo systemctl daemon-reload
sudo systemctl restart forgejo
sudo grep -E '^(DOMAIN|ROOT_URL|SSH_PORT)' /srv/forgejo/gitea/conf/app.ini
```

Because the whole LAN can reach it, turn off self-registration once your own account exists, by adding this to `[Container]`:

```ini
Environment=FORGEJO__service__DISABLE_REGISTRATION=true
```

Add `Environment=FORGEJO__service__REQUIRE_SIGNIN_VIEW=true` as well if public repositories shouldn't be visible to anyone on the LAN without signing in. Private repositories are hidden either way.

## HTTPS with a self-signed certificate

Forgejo can serve HTTPS itself, so no reverse proxy is needed. The steps below make Forgejo answer on `https://myserver.local/` (port 443), with plain HTTP on port 80 redirecting there. Browsers and Git will distrust the certificate until it has been added to each client's trust store, which is a one-off job per client. Not tested end-to-end yet.

### 1. Generate the certificate

The certificate only covers the names listed in `--host`, so list every name you'll use. `--duration` is in hours: 43800h is five years, so you aren't re-importing it on every client each year. (The 398-day maximum that browsers enforce applies to public certificate authorities, not to certificates you've trusted yourself.) Don't add `--ca`: Firefox refuses a CA certificate used directly as a server certificate.

```zsh
sudo podman exec -u git forgejo mkdir -p /data/gitea/https
sudo podman exec -u git -w /data/gitea/https forgejo forgejo cert --host myserver.local,myserver --duration 43800h
```

This writes `cert.pem` and `key.pem` to `/srv/forgejo/gitea/https/` on the host. Running it again overwrites both.

### 2. Change forgejo.container

Replace the `PublishPort=3000:3000` line and the `DOMAIN` and `ROOT_URL` lines with the following, and keep everything else (including `PublishPort=222:22`):

```ini
PublishPort=443:3000
PublishPort=80:3080
Environment=FORGEJO__server__DOMAIN=myserver.local
Environment=FORGEJO__server__ROOT_URL=https://myserver.local/
Environment=FORGEJO__server__PROTOCOL=https
Environment=FORGEJO__server__CERT_FILE=/data/gitea/https/cert.pem
Environment=FORGEJO__server__KEY_FILE=/data/gitea/https/key.pem
Environment=FORGEJO__server__REDIRECT_OTHER_PORT=true
Environment=FORGEJO__server__PORT_TO_REDIRECT=3080
```

Inside the container, Forgejo still listens on 3000 (now HTTPS) and runs its HTTP-to-HTTPS redirect on 3080. The host maps them to the standard ports 443 and 80, so URLs don't need a port number. If you later want other web apps on port 443, move HTTPS to a reverse proxy instead.

Restart, and swap the firewall ports:

```zsh
sudo systemctl daemon-reload
sudo systemctl restart forgejo
sudo firewall-cmd --permanent --add-service=https --add-service=http
sudo firewall-cmd --permanent --remove-port=3000/tcp
sudo firewall-cmd --reload
```

From another machine, `curl -kI https://myserver.local/` should get a response and `curl -I http://myserver.local/` should redirect to HTTPS. Also push something over SSH once, which exercises Forgejo's internal hooks under the new settings. On existing clones, point the remote at the new address with `git remote set-url origin https://myserver.local/<owner>/<repo>.git`.

### 3. Trust the certificate on each client

Copy only `cert.pem` to each client, never `key.pem`:

```zsh
scp myserver.local:/srv/forgejo/gitea/https/cert.pem myserver.pem
```

- **AlmaLinux / Fedora:** `sudo cp myserver.pem /etc/pki/ca-trust/source/anchors/ && sudo update-ca-trust`
- **Debian / Ubuntu:** `sudo cp myserver.pem /usr/local/share/ca-certificates/myserver.crt && sudo update-ca-certificates` (the `.crt` extension is required)
- **Windows:** rename it to `myserver.crt`, then in an administrator terminal run `certutil -addstore -f Root myserver.crt` (or double-click it, choose _Install Certificate_, and put it in _Trusted Root Certification Authorities_). Git for Windows uses the Windows certificate store by default, so nothing else is needed for Git.
- **macOS:** `sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain myserver.pem`
- **Firefox on Linux** keeps its own list: Settings &rarr; Privacy & Security &rarr; Certificates &rarr; View Certificates &rarr; Authorities &rarr; Import.

To trust it for Git only, for this server only, without touching the system store:

```zsh
git config --global http.https://myserver.local/.sslCAInfo ~/certs/myserver.pem
```

Don't set `http.sslVerify false` instead; that turns off checking for every server.

### Renewing

Check when it expires:

```zsh
openssl x509 -enddate -noout -in /srv/forgejo/gitea/https/cert.pem
```

To renew, repeat step 1, restart Forgejo (`sudo systemctl restart forgejo`), and import the new `cert.pem` on each client. If re-importing ever gets tedious, a reverse proxy such as Caddy with `tls internal` runs its own small certificate authority, which clients trust once while server certificates renew automatically, at the cost of another moving part.

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
