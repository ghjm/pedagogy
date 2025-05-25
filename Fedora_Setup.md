[Back to Top](README.md)

# Initial Fedora Setup

### Unbox and power on the mini PC
* connect it to power, a monitor, a keyboard, and optionally a mouse
* Plug in a bootable USB flash drive with Fedora 42 Server
* Press the power button, and at the same time, start repeatedly hitting `<Del>` on the keyboard
* The system should boot up to the firmware menu; at this point you can stop hitting `<Del>`
* Use the mouse or arrow keys to navigate to "BBS Menu"
* From the BBS Menu, use the mouse or arrow keys to navigate to the USB device
* Hit `<Enter>` to boot from the USB device
* If it fails to boot with a security error, you might need to disable Secure Boot:
    * From the firmware menu, go to Setup, then Security, then Secure Boot
	* Change the Secure Boot setting to Disabled
	* Save and exit
* From the Grub boot screen "Test this media and start" is the default
    * Allow it to do this the first time you use a new USB device, or if you suspect the USB is corrupted
    * On subsequent uses, you can skip this check by using the arrow keys to go to regular "start"


### Fedora Setup:
* Network & Hostname
	* Make sure wireless network shows up
	* Make sure the blue toggle switch is turned on
	* Select wifi network and password
	* Make sure to configure the hostname and click Apply
* Software Selection
	* Fedora Server Edition
	* Check "Container Management"
* Installation Destination
	* Automatic
	* Check "Free up space by removing or shrinking existing partitions"
	* Click "Delete All" then "Reclaim Space" to remove the preinstalled Windows OS (can reinstall later if needed)
* User Account
	* Enter your name, username & password
	* Leave "Add administrative privileges" and "Require a password" checked
* Begin Installation
	* Should take 5-10 minutes
	* Click "Reboot System" when done

### Initial setup after reboot:
* Log in using the username and password you chose
* Check that network connectivity is working
    * Run `ifconfig` and check that the wlan device is up and has a valid address on the local network
    * Run `ping 8.8.8.8` and verify that packets are exchanged.  Hit `<Ctrl+C>` to stop.
    * Run `ping google.com` and verify that packets are exchanged (this tests DNS resolution).  Hit `<Ctrl+C>` to stop.
	* If network connectivity isn't working, stop here and troubleshoot before continuing
* Configure passwordless sudo
	* `sudo visudo`
	* Sudo will prompt for your password
	* You will be put into the vim editor, with the `/etc/sudoers` file open
	* Use the arrow keys to scroll down to the line `%wheel ALL=(ALL) ALL`
	* With the cursor at the beginning of this line:
		* Hit `i` to go into insert mode
		* Type `# ` to comment out the line
		* Hit `<Esc>` (the escape key) to exit back to vim's normal mode
		* Hit down arrow three times, and left arrow as needed, to go to the beginning of the `%wheel ALL=(ALL) NOPASSWD: ALL` line
		* Hit `<Del>` twice to uncomment this line
		* Type `:wq<Enter>` (write quit) to save and exit
* Configure multicast DNS
	* Multicast DNS will allow you to reach this machine on the network using its name rather than having to know its IP address
	* Run `hostnamectl` and verify that the static hostname is what you entered during setup
		* If it isn't, then run `hostnamectl hostname [hostname]` to change it (replace `[hostname]` with what you want the hostname to be)
	* Run `sudo mkdir -p /etc/systemd/resolved.conf.d` to create this directory
	* Run `sudo vim /etc/systemd/resolved.conf.d/50-mdns.conf` to open this file in vim
	* Edit the file:
		* Hit `i` to go into vim insert mode
		* Type `[Resolve]` and hit `<Enter>`
		* Type `MulticastDNS=yes`
		* Hit `<Esc>` to go back to vim normal mode
		* Type `:wq<Enter>` to save and exit
	* Run `sudo systemctl restart systemd-resolved` to restart the systemd-resolved service
    * Allow incoming multicast DNS requests through the local firewall:
        * Run `sudo firewall-cmd --permanent --add-service=mdns`
        * Run `sudo firewall-cmd --reload`
	* Run `ping [hostname].local` to test pinging by name locally.  If it works, you should see 64 byte packets.  Hit `<Ctrl+C>` to stop.
	* From some other Linux box on the same LAN, do `ping [hostname].local` to make sure MDNS is working remotely.
* Configure ssh keys
    * These steps are done from your main Linux box, not from the new box
	* You should already have an ssh key
		* Run `ls -la ~/.ssh` to check
		* Personal ssh keys are two files named `id_*` and `id_*.pub` (where `*` can be `rsa`, `ed25519` or a few others)
		* The `id_*.pub` file is the public key, which can be shared; the `id_*` file is the private key, which must be kept secret
		* If you don't have an ssh key, run `ssh-keygen` to create one
	* Run `ssh [username]@[hostname].local` to connect to the new machine
	* You should be prompted for a password, and then get a bash session
	* Run `exit` or hit `<Ctrl+D>` to exit the ssh session; you are now back at your local machine's bash prompt
	* Run `ssh-copy-id [username]@[hostname].local` to copy and authorize your public key to the new machine
	* You should be prompted for a password, and get the message "Number of key(s) added: 1"
	* Now run `ssh [username]@[hostname].local` again; this time you should not be prompted for a password

### Verify correct operation of podman
* Either log in at the local console or ssh to the new machine
* Run `podman run -it alpine sh`
    * You might get messages about pulling the alpine imag from docker.io, and should then get a root prompt `/ #`
    * Run `cat /etc/os-release` and verify that it shows as Alpine Linux (not Fedora)
    * Run `exit` or hit `<Ctrl+D>` to exit
    * Run `podman ps -a` and verify the alpine container exists and shows a status of "Exited"
    * Run `podman rm [id]` where `[id]` is the first few characters of the ID shown from the previous command
    * Run `podman ps -a` again and verify the list is now empty
