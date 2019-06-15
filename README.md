# Odroid HC1 Setup
This repo contains code snippets and notes around setting up an Odroid HC1 as a NAS running Ubuntu Server 18.04. We loosely follow [instructions](https://wiki.odroid.com/odroid-xu4/software/ubuntu_nas/ubuntu_nas).

Keep in mind that this guide is more for my LAN at home and might not contain the right steps for your setup. I just wanted to be able to reproduce my setup quickly in case I need a fresh installation at some point. Also I want to document a few learnings and resources that might help out a few beginners just like me.

The HC1 does not have HDMI or any other display connector. So you will need another computer the ssh into the HC1 after [initial setup](#initial-setup)

## Hardware
- Odroid HC1
- 5V 5A PSU with 2.5mm x 5.1mm barrel plug. You should use more amps if you want to attach another HDD via USB for example
- 128GB SanDisk MicroSD card
- [Crucial MX100](https://www.crucial.com/usa/en/storage-ssd-mx100) with 500GB
- Network cable

## Initial setup
Download latest Ubuntu image from the Odroid [website](https://wiki.odroid.com/odroid-xu4/os_images/linux/ubuntu_4.14/ubuntu_4.14). Insert SD card (128GB) into a reader and flash the image using [Edger](https://www.balena.io/etcher/). Make sure to use Edger, have tried with other software but that didn't work. Also, be aware that the HC1 can only boot from microSD and not your HDD.

You can then insert the HDD into its slot and fix with the screw that comes with the HC1. Connect router and HC1 one with the network cable. Insert SD card. Connect to power and wait for the device to boot. The power light should be lit up red constantly. The green HDD light should blink a few times initially. The blue light for the microSD card should be blinking like a heartbeat. After the initial first boot, the HC1 will shut down again. Wait for it, then unplug the power and plug it in again. Now the HC1 should boot again and be up and running after maybe a minute or so.

With the HC1 up and running, we can ssh into the device. First, we need to find it's address on our network. [Angry IP Scanner](https://angryip.org/) scans your network and works across all platforms whether you are on Linux or Mac or Windows. On Linux, you can also use `nmap` or `arp-scan`. Once you have located the HC1 just ssh into it. [Credentials](https://wiki.odroid.com/odroid-xu4/os_images/linux/ubuntu_4.14/20181203-minimal) are user: `root` and password `odroid`. Assume the device address is 192.168.0.123 enter ```shh root@192.168.0.123``` into your terminal, enter `odroid` when prompted for the password. Now you should be in a shell on the odroid. Congratulations: your HC1 is up and running!

For the remainder of the guide, we will assume to be working in the shell. If not, we will say so explictly. Also, until we have set up new users, we assume that you are logged in as `root`.

## Update image
As root, run
```apt update && apt full-upgrade -y```
to generate a fresh package index. While running that you will get prompted that the old `boot.ini` has been saved to `/media/boot/boot.ini.old`. If it shows a different location adjust next step accordingly. Also, you will be asked whether services should be restarted automatically. Hit `yes`.

Remove old boot.ini:
```rm /media/boot/boot.ini.old```

Let's update our timezone and locale:
```dpkg-reconfigure tzdata```
and answer as appropriate. Then, we set our locale using
```
locale-gen "en_US.UTF-8"
update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
dpkg-reconfigure locales
```
You can basically confirm through that without changes unless you need different locales.

Initial setup is done, so let's ```reboot```. Congratulations: You now have a clean image!

## Static IP address
We want a static IP address if to use the HC1 as a NAS. Ubuntu 18.04 uses [netplan](https://netplan.io/) for configuring networks. We will use [NetworkManager](https://help.ubuntu.com/community/NetworkManager).

It should be installed but let's check:
```
dpkg -l | grep network-manager
```

Also let's check for if there is a config already:
```
ll /etc/netplan
```

If not, we create a new one and open it:
```
touch /etc/netplan/netconf.yaml
nano /etc/netplan/netconf.yaml
```

Check for name of your ethernet connection:
```
ip a
```

Copy/paste the `netconf.yaml` from this repo which is veeeerrrry simple. We have opted to use Cloudflare's public [DNS](https://www.cloudflare.com/learning/dns/what-is-dns/) and n. Make adjustments as needed. In particular, update config if it is not `eth0`. Save config and try to apply.
```netplan try```

If it works, this should kill you ssh connection. Wait a bit and try to connect to new address:
```ssh root@192.168.0.2```
or whatever static address you have assigned. Congratulations, you now have a static IP address!

## Hostname
This [guide](https://www.cyberciti.biz/faq/ubuntu-18-04-lts-change-hostname-permanently/) helps you to change your hostname.

```
hostnamectl set-hostname yourname
nano /etc/hosts
```
Here, you change `odroid` to `yourname`. Then:
```
reboot
```

## Welcome message
We will do that later. This [guide](https://linuxconfig.org/how-to-change-welcome-message-motd-on-ubuntu-18-04-server) might help.

## Root and sudo user
First, let's change the root password from `odroid` to a different one.

```
passwd
```

Then, we create a sudo user, so that we can remove ssh access for root. Add the user and add it to the sudo group:
```
adduser your-user-name
usermod -G sudo your-user-name
```
Exit the shell with `Ctrl+D` and try to shh in with your new user. Perfect. Log out again and ssh in as root.

Now, we can disable ssh access for the root user:
```
nano /etc/ssh/sshd_config
```

Change "PermitRootLogin" to **no**, save and restart the ssh daemon:
```
service sshd restart
```
Log out of the shell and log in as sudo user from now on.

## Filesystem
