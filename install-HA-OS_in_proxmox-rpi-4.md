# Install Raspberry Pi OS **64bit**

Use ethernet for the whole process.

A detailed guide can be found [here](https://pycvala.de/blog/raspberry-pi/raspberry-pi-installing-proxmox-ve-7-on-the-pi-4/#what-youll-need) but the summary of the steps are:

* download the rpi os 64 bit desktop from [here](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2022-09-26/2022-09-22-raspios-bullseye-arm64.img.xz)
* use balenaEtcher or raspberry pi imager to put it in an SD card
* enable ssh: create an empty and without extension file with name "ssh" (no quotes) in the boot folder of your created sd card

You can do everything from ssh but its easier to use a keyboard/mouse/display connected to the rpi.

# Install Pimox and create VM
* as sudo install pimox with the info [here](https://github.com/pimox/pimox7) (automatic installer worked fine)
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

