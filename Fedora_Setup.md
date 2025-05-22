* Boot from Fedora 42 Server USB device
	* If it fails to boot, you might need to disable Secure Boot:
		* Press Del repeatedly while powering on to get to BIOS Setup menu
		* Go to Security - Secure Boot and disable it
		* Save and exit
	* Choose "Test this media and start" the first time you use a new USB device, then just regular "start" afterwards

* Fedora Setup:
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

* Initial setup after reboot:
	* Log in using the username and password you chose
	* Run `ifconfig` and check that the wlan device is up and has a 10.1.1.* IPv4 address
		* If it does not, stop here and troubleshoot the wifi before continuing
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
		* Run `sudo vim /etc/system/resolved.conf.d/50-mdns.conf` to open this file in vim
		* Edit the file:
			* Hit `i` to go into vim insert mode
			* Type `[Resolve]` and hit `<Enter>`
			* Type `MulticastDNS=yes`
			* Hit `<Esc>` to go back to vim normal mode
			* Type `:wq<Enter>` to save and exit
		* Run `sudo systemctl restart systemd-resolved` to restart the systemd-resolved service
		* Run `ping [hostname].local` to test pinging by name.  If it works, you should see 64 byte packets.  Hit `<Ctrl+C>` to stop.
		* From another machine on the network, try `ping [hostname].local`
	* Configure ssh keys
		* On your main Linux box, you should already have an ssh key
			* Run `ls -la ~/.ssh` to check
			* Personal ssh keys are two files named `id_*` and `id_*.pub` (where `*` can be `rsa`, `ed25519` or a few others)
			* The `id_*.pub` file is the public key, which can be shared; the `id_*` file is the private key, which must be kept secret
			* If you don't have an ssh key, run `ssh-keygen` to create one
		* Run `ssh [username]@[hostname].local` to connect to the new machine
		* You should be prompted for a password, and then get a bash session
		* Run `logout` or hit `<Ctrl+D>` to exit the ssh session; you are now back at your local machine's bash prompt
		* Run `ssh-copy-id [username]@[hostname].local` to copy and authorize your public key to the new machine
		* You should be prompted for a password, and get the message "Number of key(s) added: 1"
		* Now run `ssh [username]@[hostname].local` again; this time you should not be prompted for a password
