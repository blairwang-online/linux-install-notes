# Configure NTS time sync (chrony)

Keep the clock in sync with Network Time Security (NTS), which is NTP with authentication, so the machine can't be fed a fake time by anyone between it and the time server. Plain NTP has no such protection.

I use two independent operators:

- **`ntspool.time.nl`** (TimeNL, run by SIDN in the Netherlands): a pool of four stratum 1 servers, so it goes in a `pool` line and chrony uses up to four of them.
- **`nts.netnod.se`** (Netnod, Sweden): one name, anycast across Netnod's Swedish sites and offered for users anywhere in the world, so it goes in a `server` line.

Both are free to use from anywhere. Both are in Europe, so from Australia each request makes a round trip of roughly a third of a second. NTP copes with that well, but the clock ends up within a few milliseconds of UTC rather than within a fraction of one. That's plenty for desktops, servers, TLS and logs. `chronyc tracking` (see [Check that it works](#check-that-it-works)) shows the real figure.

This needs **chrony**: `systemd-timesyncd` doesn't support NTS. Outbound **UDP 123** (NTP) and **TCP 4460** (the NTS key exchange) must be open; firewalls normally allow outbound traffic, so there is nothing to do unless the network blocks it.

Drafted with AI assistance; not tested end-to-end yet.

The two source lines are the same on every distro:

```
# NTS (authenticated NTP). Needs outbound UDP 123 and TCP 4460.
pool ntspool.time.nl iburst nts
server nts.netnod.se iburst nts
```

## AlmaLinux, Fedora and Fedora Silverblue

chrony is installed and running by default, configured in `/etc/chrony.conf`. (`/etc` is writable on Silverblue too, so the steps are the same.)

```bash
# Keep a copy of the original
sudo cp /etc/chrony.conf /etc/chrony.conf.orig

# Comment out the default pool (e.g. 2.almalinux.pool.ntp.org) and the
# DHCP-supplied servers (sourcedir /run/chrony-dhcp), which aren't authenticated
sudo sed -i -E 's/^(pool|server|sourcedir) /#&/' /etc/chrony.conf

# Add the NTS sources
sudo tee -a /etc/chrony.conf > /dev/null << 'EOF'

# NTS (authenticated NTP). Needs outbound UDP 123 and TCP 4460.
pool ntspool.time.nl iburst nts
server nts.netnod.se iburst nts
EOF

# chrony should save its NTS cookies, so a restart doesn't redo the key exchange.
# The default file already has this line; add it if this prints nothing.
grep '^ntsdumpdir' /etc/chrony.conf

sudo systemctl restart chronyd
```

## Ubuntu 26.04 LTS

Ubuntu has used chrony with NTS by default since 25.10, with Ubuntu's own servers listed in `/etc/chrony/sources.d/ubuntu-ntp-pools.sources`. To use the servers above instead:

```bash
# Add the NTS sources
sudo tee /etc/chrony/sources.d/nts.sources > /dev/null << 'EOF'
# NTS (authenticated NTP). Needs outbound UDP 123 and TCP 4460.
pool ntspool.time.nl iburst nts
server nts.netnod.se iburst nts
EOF

# Switch off Ubuntu's sources (chrony only reads files ending in .sources)
sudo mv /etc/chrony/sources.d/ubuntu-ntp-pools.sources /etc/chrony/sources.d/ubuntu-ntp-pools.sources.disabled

# Don't use DHCP-supplied servers, which aren't authenticated
sudo sed -i 's|^sourcedir /run/chrony-dhcp|#&|' /etc/chrony/chrony.conf

sudo systemctl restart chrony
```

Ubuntu's file also listed `ntp-bootstrap.ubuntu.com`, which helps a machine whose clock is far off get started. Without it, see [If the clock is far off](#if-the-clock-is-far-off).

A machine upgraded from 24.04 may still be running `systemd-timesyncd`. Check with `systemctl is-active chrony systemd-timesyncd`. If timesyncd is the active one, follow the 24.04 steps below.

## Ubuntu 24.04 LTS and Debian

These use `systemd-timesyncd`, which can't do NTS, so install chrony. apt removes `systemd-timesyncd` at the same time.

```bash
sudo apt install chrony

# Add the NTS sources
sudo tee /etc/chrony/sources.d/nts.sources > /dev/null << 'EOF'
# NTS (authenticated NTP). Needs outbound UDP 123 and TCP 4460.
pool ntspool.time.nl iburst nts
server nts.netnod.se iburst nts
EOF

# Comment out the default pools and the DHCP-supplied servers
sudo sed -i -E 's/^(pool|server) /#&/; s|^sourcedir /run/chrony-dhcp|#&|' /etc/chrony/chrony.conf

# chrony.conf must read /etc/chrony/sources.d. If this prints nothing,
# add the two source lines to /etc/chrony/chrony.conf itself instead.
grep '^sourcedir /etc/chrony/sources.d' /etc/chrony/chrony.conf

sudo systemctl restart chrony
```

## Check that it works

Give it a minute after restarting, then:

```bash
# Sources: the one marked ^* is in use. Expect time.nl and netnod.se names.
chronyc -N sources

# Every source should show Mode NTS, with non-zero KeyID, Type and KLen
sudo chronyc -N authdata

# "System time" and "RMS offset" show how close the clock is to UTC
chronyc tracking

# Expect "System clock synchronized: yes" and "NTP service: active"
timedatectl status
```

## Troubleshooting

### If the clock is far off

NTS starts with a TLS handshake, and TLS checks that the server's certificate is valid *now*. If the machine's clock is far off (a new machine, a flat CMOS battery), the certificates look expired or not yet valid, and chrony never syncs. Set the clock roughly by hand, then restart chrony:

```bash
sudo date -s "2026-11-22 10:00"
sudo systemctl restart chronyd   # Ubuntu/Debian: chrony
```

Alternatively, add `nocerttimecheck 1` to the chrony config, which skips the certificate date check for the first clock update only. It's a small security trade-off, so remove it once the clock is right.

### If the sources never become reachable

If `chronyc -N sources` shows the servers but `sudo chronyc -N authdata` shows no keys, the network is probably blocking TCP port 4460. Some guest and corporate networks do. chrony can't sync over NTS from such a network.

## Sources

- [Netnod: Network Time Security](https://www.netnod.se/nts/network-time-security)
- [TimeNL: NTS servers](https://nts.time.nl/)
- [Ubuntu: Synchronize time using chrony](https://ubuntu.com/server/docs/how-to/networking/chrony-client/)
- [chrony.conf(5)](https://chrony-project.org/doc/latest/chrony.conf.html)
