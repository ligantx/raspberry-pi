# Install Raspberry Pi OS **64bit**

Use ethernet for the whole process.

A detailed guide can be found [here](https://pycvala.de/blog/raspberry-pi/raspberry-pi-installing-proxmox-ve-7-on-the-pi-4/#what-youll-need) but the summary of the steps are:

* download the rpi os 64 bit desktop from [here](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2022-09-26/2022-09-22-raspios-bullseye-arm64.img.xz)
* use balenaEtcher or raspberry pi imager to put it in an SD card
* enable ssh: create an empty and without extension file with name "ssh" (no quotes) in the boot folder of your created sd card

# Install Pimox and create HA OS VM
* as sudo install pimox with the info [here](https://github.com/pimox/pimox7) (automatic installer worked fine)
* if everything is fine you can enter Pimox with your_ip_address:8006 (login_user: root, password: your password)
* Make sure under your node in System and network, you have eth0 network device **and** a linux bridge already there.
* Create a new VM (change only the following, the rest are default):
  * Use VM ID: 100
  * OS: do not use any media
  * BIOS: UEFI
