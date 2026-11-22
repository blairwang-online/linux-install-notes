# Intel N100 mini PC as a low-power server

Notes for running a small, fanless Intel N100-class (or N150) mini PC as an always-on, low-power server, here with 8 GB of RAM, onboard eMMC plus an M.2 NVMe SSD, and USB-C Power Delivery input. The OS is AlmaLinux 10 set up as a [workstation/server hybrid](../distros/almalinux.md#workstationserver-hybrid-el10), serving [Samba shares](setup-samba.md) and [Forgejo](forgejo-podman.md). Most of this applies to similar mini PCs and other distros too. Not tested end-to-end yet.

## BIOS / UEFI settings

- **Firmware TPM:** turn on Intel PTT, so LUKS can unlock with [TPM + PIN](../distros/almalinux.md#luks-unlock-with-tpm--pin-and-back-up-the-header).
- **Power on after power loss:** turn on "restore on AC power loss" (or similar wording), so the machine comes back by itself after a power cut.
- **Secure Boot:** leave it on. AlmaLinux supports it.

## Storage

- Choose a **DRAM-less TLC** NVMe SSD. DRAM-less drives use a little system RAM for their tables (Host Memory Buffer, usually tens of MB, which is nothing on 8 GB) and run cooler and use less power. Avoid QLC drives where you can, and avoid PCIe Gen5 drives, which run hot.
- Put a **thermal pad** between the SSD and the case lid. In a fanless box the case is the heatsink.
- **Install to the NVMe drive only.** The installer also lists the eMMC (`mmcblk0`) and the USB stick; untick both. `lsblk` tells them apart: `nvme0n1` is the SSD.
- **Use the eMMC as an on-box backup target** rather than leaving it idle: for example, LUKS + XFS mounted at `/backup`, with a nightly `restic` or `rsync` copy of `/srv` (Forgejo's data) and wherever the Samba shares live. It protects against the SSD failing, but not against losing the whole machine, so keep an off-box backup as well.
- Keep an eye on eMMC wear if it's written to regularly: `sudo mmc extcsd read /dev/mmcblk0 | grep -i life` (from `mmc-utils`).

## Living with 8 GB of RAM

- Turn off **KDUMP** in the installer.
- About 4 GiB of swap is plenty.
- No virtual machines (skip Cockpit's Machines app). Containers are fine.
- Run GNOME only when a display is plugged in (see [boot to the console](../distros/almalinux.md#boot-to-the-console-start-gnome-only-when-needed)).

## Wi-Fi chip

The N100 supports x86-64-v3, so the standard AlmaLinux 10 `x86_64` ISO is the right one. The Wi-Fi card varies between models, though, so check it after installing:

```zsh
lspci -nnk | grep -A3 -i network
```

- **Intel** (driver `iwlwifi`): fine, the driver is in the RHEL kernel.
- **Realtek or MediaTek:** RHEL kernels leave out many of these drivers. [ELRepo](https://elrepo.org) packages some of them as `kmod-*` packages, but kernel updates can arrive a few days before the matching kmod does. If the machine depends on Wi-Fi, consider Ethernet, a different card, or Debian.

## Power checks after installing

Some SSDs stop the CPU reaching its deeper sleep states, which raises idle power for the whole machine. Check while you can still return the drive:

```zsh
sudo dnf install -y nvme-cli powertop smartmontools lm_sensors

# PCIe power saving policy (shouldn't be "performance")
cat /sys/module/pcie_aspm/parameters/policy

# NVMe autonomous power state transitions (APST) should be enabled
sudo nvme get-feature -f 0x0c -H /dev/nvme0

# Leave this running for a few minutes at idle: on the "Idle stats" tab,
# the CPU package should spend time in C6 or deeper, not just C2/C3
sudo powertop

# Temperatures
sudo smartctl -a /dev/nvme0 | grep -i temp
sensors
```

A cheap USB-C power meter between the charger and the machine settles it: compare idle watts with and without the SSD. An extra half a watt is fine; 2 W or more means the drive is blocking power saving.

Then set the power profile:

```zsh
sudo tuned-adm profile powersave
```

## Power and heat

- Check which USB-C PD voltages the machine accepts (often 12–20 V) and use a charger that offers them. Phone chargers that only offer 5 V or 9 V usually won't run it.
- Fanless boxes run warm. Give the case some air, and don't run heavy jobs while it's packed in a bag.
