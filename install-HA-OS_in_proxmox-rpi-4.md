# Install Raspberry Pi OS **64bit**

Use ethernet for the whole process.

A detailed guide can be found [here](https://pycvala.de/blog/raspberry-pi/raspberry-pi-installing-proxmox-ve-7-on-the-pi-4/#what-youll-need) but the summary of the steps are:

* download the rpi os 64 bit desktop from [here](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2022-09-26/2022-09-22-raspios-bullseye-arm64.img.xz)
* use balenaEtcher or raspberry pi imager to put it in an SD card
* enable ssh: create an empty and without extension file with name "ssh" (no quotes) in the boot folder of your created sd card

You can do everything from ssh but its easier to use a keyboard/mouse/display connected to the rpi.

# Restore from promox backup

* The promox backups are stored in `/var/lib/vz/dump`. Copy the file there somewhere safe.
* Install Raspberry Pi OS 64bit and install promox (node: raspberry, ip: the same as the one you ssh to raspberry).
* Copy the backup files in `/var/lib/vz/dump` again. From Promox UI go to Datacenter, Rapsberry (the name of your node), local (your storage) and Backups, where you will see your saved backup.
* Restore the backup, and run the VM. Wait..
* If HA ipv4 address is not accessible try ipv6, or ipv4 routing. 

# Install Pimox and create VM
* as sudo install pimox with the info [here](https://github.com/pimox/pimox7) (automatic installer worked fine). Do not change `/etc/dhcpcd.conf` file. During installation insert the ip raspberry pi has already.
* if everything is fine you can enter Pimox with `your_raspberry_pi_ip_address:8006` (login_user: root, password: your password)
* Make sure under your node in System and network, you have eth0 network device **and** a linux bridge already there.
* Create a new VM (change only the following, the rest are default):
  * Use VM ID: 100
  * OS: do not use any media
  * BIOS: UEFI, efi storage: local, qemu image format
  * cpu: cores 2, **type: host**
  * memory: can be 4096 but depending on your rpi total ram
 
 # Connect HA OS with the VM
 
* **This is important**: For raspberry pi you have to download **aarch64** HA OS image from [here](https://github.com/home-assistant/operating-system/releases/download/9.3/haos_generic-aarch64-9.3.qcow2.xz).

* Unzip the `haos_generic-aarch64-9.3.qcow2.xz`. Now you will have `haos_generic-aarch64-9.3.qcow2` file.

* With terminal go to the `haos_generic-aarch64-9.3.qcow2` file and import your file to the VM like [this video](https://youtu.be/PrKQkI53xys?t=831) (use `./haos_generic-aarch64-9.3.qcow2` and maybe `local` instead of `local-lvm`. 

* As the video, detach your VM hard disk drive, delete this drive, and add your imported disk.
* **delete** CD/DVD (ide2) drive
* Go to options and boot order, enable and put first the scsi disk.
* Run the VM and **wait** until you see the banner (the one that shows the HA ip address). THIS ip is the one you'll use to enter HA and **not** the one of raspberry pi os host.
* Install [guest agent](https://pycvala.de/blog/raspberry-pi/raspberry-pi-installing-proxmox-ve-7-on-the-pi-4/#what-youll-need)
* Now, you can restore your snapshots from previous HA as usual (you can follow the previous video for more).

# HA OS FIXES

* USB devices cannot be used from guest HA OS proxmox VM (maybe a bug in pimox?). So, if you have for example a Sonoff Zigbee 3 usb stick, you should follow the info found [here](https://github.com/pimox/pimox7/issues/48#issuecomment-1065759910) you can fix it:

1. Go to Proxmox GUI and your device Shell (under Datacenter) and write the following:
`vim /etc/pve/qemu-server/<VMID>.conf`. For example: `vim /etc/pve/qemu-server/100.conf`
2. go to the end of the file and press `o` and paste the following:
`args: -device qemu-xhci -device usb-host,vendorid=<0xVendorid>,productid=<0xProductid>`
you'll find vendor and product id from Hardware page of the VM in proxmox GUI, if we try to add a usb device.
for example it can be:
`args: -device qemu-xhci -device usb-host,vendorid=0x11b2,productid=0xjk30`
3. restart the VM
4. go inside the VM and find your zigbee device as usual, or run Terminal add on and write: `ls /dev/serial/by-id` and your device should apear.


* In case you use Frigate: it will not work if you have hardware accelerators: You have to delete `ffmpeg: hwaccel_args: ` lines in `frigate.yml`

# TAILSCALE

1. Install it with Tailscale add on in HA.
2. From the tailscale admin panel enable `subnet routes` for example `192.168.X.0/24` (maybe you'll need to find more info about the installation in Tailscale's page). 
3. Try to go out of the network and enter 192.168.X.X:8123 (your HA ip) as usual. I found that if raspberry pi (proxmox vmbr0) has the same ip with HA it makes tailscale unstable. Better change the vmbr0 ip from proxmox node network page.
4. You can also enable exit node in tailscale admin page. So, if you enable it in e.g. android client, you can enter every page for example `google.com` using your vpn (maybe useful if you are in a cafeteria), but makes your connection slower.

# SEAFILE

Easiest install with docker from [here](https://manual.seafile.com/docker/deploy_seafile_with_docker/)

Nice info video [here](https://www.youtube.com/watch?v=gQ1WYgy6Z8s&t=265s)

1. `sudo apt-get install docker-compose -y`
2. `mkdir ~/docker/seafile && cd ~/docker/seafile`
3. copy the yml file from [here](https://download.seafile.com/d/320e8adf90fa43ad8fee/files/?p=/docker/docker-compose.yml)
and paste it here:
`vim docker-compose.yml` and save with :wq
4. change `MY_ROOT_PASSWORD` and `DB_ROOT_PASSWD` to your password, `SEAFILE_ADMIN_EMAIL` to your email and `SEAFILE_ADMIN_PASSWORD` to your password
5. db volumes to:
`- /home/YOUR_USER/docker/seafile/seafile-mysql/db:/var/lib/mysql`
ports to:
`- "8080(or your port of preference):80"`
seafile volumes to:
`- /home/YOUR_USER/docker/seafile/seafile-data:/shared`

6. Enter your seafile admin page:
`http://rpi-ip:8080` and enter with your admin email and password

7. If you want to change admin password in terminal run (info [here](https://forum.seafile.com/t/reset-admin-password/15807)):
`docker exec -it seafile /opt/seafile/seafile-server-latest/reset-admin.sh`

8. if you want to see your seafile files directly you can use [seafile fuse](https://manual.seafile.com/extension/fuse/#how-to-start-seaf-fuse-in-docker)

# Extra Resources

[https://gist.github.com/luckydonald/1849291fb5e19c87df8c8a1618e29eaa#pimox](https://gist.github.com/luckydonald/1849291fb5e19c87df8c8a1618e29eaa#pimox)
