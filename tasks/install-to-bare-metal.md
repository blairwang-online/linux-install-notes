# Install to Bare Metal

- Avoid secure boot if you intend to use VirtualBox - unless you are absolutely sure that it is able to work.
- If installing on a machine that previously had Linux, especially if you are dual-booting with Windows, the Anaconda installer might crash due to `/boot/efi` being full. If this happens:
	- Boot from the Live USB again, if you can. (If you can't, you might need to rebuild the Live USB.)
	- Use <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>1</kbd> (or was it 2, or 3...?) to get access to the terminal.
	- `mkdir ~/boot-efi` to give yourself a mountpoint
	- `lsblk` to list the partitions
	- Use `mount` to mount the correct partition to `~/boot-efi`
	- Then you can use the terminal to go in and delete the stuff inside `/boot/efi` left over from the previous Linux installation. In my case, I was switching from Fedora to AlmaLinux, so I deleted all the old Fedora stuff.
