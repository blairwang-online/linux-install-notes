# Forgejo in a Podman container

Self-hosted Git forge on AlmaLinux 10, run as a rootful Podman Quadlet so systemd starts it at boot, and served over HTTPS from the very first start with a self-signed certificate. Adapted from the Podman Quadlet example in https://forgejo.org/docs/latest/admin/installation-docker/ (tested by Forgejo on AlmaLinux 9.4 with Podman 5.2). Not tested end-to-end here yet.

The order matters: set up the hostname, create the certificate, start Forgejo with HTTPS already on, trust the certificate on your own machine, and only then open the setup page. Forgejo serves HTTPS only, on port 3000, and never uses ports 80 or 443, so those stay free for other applications on the same machine.

## Which version

Use the LTS tag. Forgejo 15 is the current LTS, supported until 15 July 2027, and the `15` tag always points at the latest 15.x release. The next LTS is expected to be 19 (April 2027). Moving between major versions (for example 15 to 19) is a manual step: read the release notes and back up first.

Forgejo 15's container images trust reverse-proxy headers from anywhere by default (`REVERSE_PROXY_TRUSTED_PROXIES = *`; changed in 16, but not in the 15 LTS). That only matters behind a reverse proxy, but the configuration below narrows it to localhost anyway. It controls which addresses Forgejo trusts to pass on headers such as `X-Forwarded-For`, not who can connect, so don't widen it to the LAN.

## 1. Set up the hostname

`DOMAIN` and `ROOT_URL` decide the links Forgejo generates: clone URLs, redirects after sign-in, and links in emails and webhooks. Forgejo answers on any address, but if you browse by an address other than `ROOT_URL`, it shows a warning banner and some links point to the `ROOT_URL` address. The certificate will also only be valid for the names it was made for. So settle on a hostname that every network you use can resolve before going further.

With mDNS, `myserver.local` works on most LANs without any router changes, which suits a machine that moves between networks:

```zsh
sudo dnf install -y avahi
sudo systemctl enable --now avahi-daemon
sudo firewall-cmd --permanent --add-service=mdns
sudo firewall-cmd --reload
```

Recent Windows, macOS and most Linux desktops resolve `.local` names. A VPN name that doesn't change (WireGuard, Tailscale) works the same way.

The rest of this page uses `myserver.local` (and `myserver` as a second name in the certificate). Replace them throughout.

## 2. Create the certificate

Generate a self-signed certificate with Forgejo's own `forgejo cert` command, in a throwaway container, before Forgejo runs for the first time:

```zsh
sudo dnf install -y podman
sudo mkdir -p /srv/forgejo/gitea/https
sudo chown -R 1000:1000 /srv/forgejo/gitea
sudo podman run --rm --user 1000:1000 -v /srv/forgejo:/data:Z -w /data/gitea/https --entrypoint forgejo codeberg.org/forgejo/forgejo:15 cert --host myserver.local,myserver --duration 43800h
ls -l /srv/forgejo/gitea/https
```

The last command should list `cert.pem` and `key.pem`.

- `--host` takes a comma-separated list of every name you'll use. Browsers reject the certificate for any address not on that list.
- `--duration` is in hours: 43800h is five years, so you aren't re-importing it on every client each year. The 398-day maximum that browsers enforce applies to public certificate authorities, not to certificates you've trusted yourself.
- Don't add `--ca`: Firefox refuses a CA certificate used directly as a server certificate.
- UID 1000 is the `git` user inside the container, matching `USER_UID` in step 3. On the host, it's normally the first user created at install.

## 3. Create the Quadlet files

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
Environment=FORGEJO__server__PROTOCOL=https
Environment=FORGEJO__server__CERT_FILE=/data/gitea/https/cert.pem
Environment=FORGEJO__server__KEY_FILE=/data/gitea/https/key.pem
Environment=FORGEJO__server__DOMAIN=myserver.local
Environment=FORGEJO__server__ROOT_URL=https://myserver.local:3000/
Environment=FORGEJO__server__SSH_PORT=222
Environment=FORGEJO__session__COOKIE_SECURE=true
Environment=FORGEJO__security__REVERSE_PROXY_TRUSTED_PROXIES=127.0.0.0/8,::1/128

[Service]
Restart=always

[Install]
WantedBy=default.target
```

Notes on these settings:

- Forgejo serves HTTPS only (`PROTOCOL=https`) on port 3000, so its address is `https://myserver.local:3000/`. Ports 80 and 443 stay free for other applications. There's no HTTP-to-HTTPS redirect: Forgejo can only offer one on a separate port, and that's only useful on port 80. Typing `http://myserver.local:3000/` gets an error page saying the client sent an HTTP request to an HTTPS server; use `https://`.
- `COOKIE_SECURE=true` makes browsers send the sign-in cookie over HTTPS only.
- Port 222 is for Git over SSH, leaving 22 for the host's own SSH. `SSH_PORT=222` makes Forgejo show `:222` in its clone URLs; inside the container, SSH still listens on 22.
- `/srv/forgejo` holds everything: repositories, the SQLite database, `gitea/conf/app.ini`, and the certificate and its private key. `:Z` gives it the SELinux label the container needs. Don't use `:Z` on system folders or anything under `/home`.
- `FORGEJO__section__KEY` variables are written into `app.ini` each time the container starts. They can add or change settings but not remove them; edit `app.ini` for that.
- If a reverse proxy later takes ports 80 and 443 for several applications, Forgejo can sit behind it: point the proxy at `https://127.0.0.1:3000` and change `ROOT_URL` to the proxy's address.

## 4. Start Forgejo and open the firewall

```zsh
sudo systemctl daemon-reload
sudo systemctl start forgejo
sudo firewall-cmd --permanent --add-port=3000/tcp --add-port=222/tcp
sudo firewall-cmd --reload
```

`firewall-cmd` refuses to mix `--add-service` and `--add-port` in one command (`argument --add-port: not allowed with argument --add-service`), so if you add services at the same time, use a separate command for them.

Don't run `systemctl enable forgejo`. Quadlet generates the service, and the `[Install]` section is what starts it at boot; `enable` fails on a generated unit.

From another machine (the server itself may not resolve its own `.local` name), check that `curl -kI https://myserver.local:3000/` gets a response. Don't open the setup page yet.

## 5. Trust the certificate on your clients

Do this at least on the machine you'll use for the setup page, so the browser shows a normal padlock rather than a warning you'd have to click through. Copy only `cert.pem`, never `key.pem`:

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
git config --global http.https://myserver.local:3000/.sslCAInfo ~/certs/myserver.pem
```

The port has to be in that setting, because Git matches it exactly. Don't set `http.sslVerify false` instead; that turns off checking for every server.

## 6. Finish setup in the browser

Go to `https://myserver.local:3000/` and finish setup straight away: until it's done, anyone who can reach the server can complete it. The server and URL fields are already filled in from `forgejo.container`; leave them as they are. SQLite (the default) is fine for a small personal server. Create the admin account there.

Because the whole LAN can reach Forgejo, then turn off self-registration by adding this to `[Container]` in `forgejo.container`:

```ini
Environment=FORGEJO__service__DISABLE_REGISTRATION=true
```

Add `Environment=FORGEJO__service__REQUIRE_SIGNIN_VIEW=true` as well if public repositories shouldn't be visible to anyone on the LAN without signing in. Private repositories are hidden either way. Then apply it and check the settings reached `app.ini`:

```zsh
sudo systemctl daemon-reload
sudo systemctl restart forgejo
sudo grep -E '^(PROTOCOL|DOMAIN|ROOT_URL|SSH_PORT|DISABLE_REGISTRATION)' /srv/forgejo/gitea/conf/app.ini
```

Once your SSH key is added in Forgejo, test both ways in:

```zsh
ssh -F /dev/null -p 222 git@myserver.local
git clone https://myserver.local:3000/<owner>/<repo>.git
```

Push something over SSH once as well, which exercises Forgejo's internal hooks.

## Who can connect

Nothing in `forgejo.container` limits who can connect. `PublishPort` listens on all of the host's network interfaces and the firewall ports are open, so any machine on the LAN can reach Forgejo. To limit the web interface to this machine instead, use `PublishPort=127.0.0.1:3000:3000`. `DOMAIN`, `ROOT_URL` and `REVERSE_PROXY_TRUSTED_PROXIES` don't control access either (see above).

## Changing the address later

1. Generate a new certificate for the new names, overwriting the old one (Forgejo can stay running): `sudo podman exec -u git -w /data/gitea/https forgejo forgejo cert --host <new names> --duration 43800h`
2. Update `DOMAIN` and `ROOT_URL` in `forgejo.container`, then `sudo systemctl daemon-reload` and `sudo systemctl restart forgejo`.
3. Import the new `cert.pem` on each client, and point existing clones at the new address: `git remote set-url origin https://<new address>:3000/<owner>/<repo>.git`

## Renewing the certificate

Check when it expires:

```zsh
openssl x509 -enddate -noout -in /srv/forgejo/gitea/https/cert.pem
```

To renew, run the `podman exec ... forgejo cert` command from [Changing the address later](#changing-the-address-later) with the same names, restart Forgejo, and import the new `cert.pem` on each client. If re-importing ever gets tedious, a reverse proxy such as Caddy with `tls internal` runs its own small certificate authority, which clients trust once while server certificates renew automatically, at the cost of another moving part.

## Updates

`AutoUpdate=registry` lets Podman pull newer 15.x images. Turn on the timer that does it:

```zsh
sudo systemctl enable --now podman-auto-update.timer
```

To update by hand: `sudo podman auto-update`.

## Backups

Everything is in `/srv/forgejo`, including the certificate's private key, so keep backups encrypted. For a consistent copy, stop the service first:

```zsh
sudo systemctl stop forgejo
# copy or snapshot /srv/forgejo
sudo systemctl start forgejo
```

## Troubleshooting

- Logs: `sudo journalctl -u forgejo` or `sudo podman logs forgejo`.
- If the container can't write to `/srv/forgejo`, check for SELinux denials: `sudo ausearch -m avc -ts recent`.
- If step 2 fails with an error about `forgejo` not being found, use `--entrypoint /app/gitea/gitea` instead (the binary's location in the image).
- Cockpit's _Podman containers_ page (`cockpit-podman`) shows the container and its logs.
