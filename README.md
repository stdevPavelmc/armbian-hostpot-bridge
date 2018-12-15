# Your SBC as a wireless router connected to your local LAN

Main goal is to create detailed instructions to turn any SBC that runs [Armbian](https://www.armbian.com/) or [Raspbian](https://www.raspbian.org/) into a wireless router with full security that allows you to use your home lan to route traffic to the internet. Main focus is on Armbian OS, but instructions for Raspbian are included to when needed.

A notice: all the [Raspberry Pis](https://www.raspberrypi.org/) that use [Raspbian](https://www.raspbian.org/) and need to do some extra steps, I will point that extra steps along the way. Of curse if you has a Raspberry Pi with no WiFi you can add a USB WiFi dongle.

## Requirements

You will need the following to complete this guide:

* Single Board Computer that supports Linux, Armbian or Raspbian [tested on Orange Pi Prime & Raspberry B+ plus USB WiFi dongle]
* uSD card, at least 8 GB and class 10 recommended.
* USB keyboard
* HDMI cable
* HDMI capable monitor
* Power adapter for the Orange Pi Prime SBC  (5.0 V @ 2A or more) or USB cable for Raspberry Pi
* A free ethernet port on your home router with internet access

I will assume that your local network has a DHCP server, this DHCP server will serve wifi users as well as we will bridge the LAN and WIFI networks.

## Step 1: Get Armbian/Raspbian and flash it

Get armbian for your SBC from [the official site](https://www.armbian.com/) pick the non desktop one as we will require not UI.

If you use a Raspberry go to official site and download the Raspbian lite version.

Once you have the images un-compress the file and flash it to a uSD card, you can find a lot of guides about how to do it over the internet.

## Step 2: Plug it and start it

Insert the uSD card into the SBC, connect the ethernet cable from your local lan, the monitor and the usb Keyboard; at the end connect the power cable/USB cable as you need.

### Step 2a: for Armbian, start it and create the user

You will see the system boot and will be presented with a login prompt, use armbian default credentials

```
user: root
password: 1234
```

Once there you will be asked change the default root password, this is a screen snipet of the process:

```
[  OK  ] Reached target Multi-User System.
[  OK  ] Reached target Graphical Interface.
         Starting Update UTMP about System Runlevel Changes...
[  OK  ] Started Update UTMP about System Runlevel Changes.

Debian GNU/Linux 9 orangepiprime ttyS0

orangepiprime login: root
Password: 
You are required to change your password immediately (root enforced)
Changing password for root.
(current) UNIX password: 
Enter new UNIX password: 
Retype new UNIX password: 
  ___                               ____  _   ____       _                
 / _ \ _ __ __ _ _ __   __ _  ___  |  _ \(_) |  _ \ _ __(_)_ __ ___   ___ 
| | | | '__/ _` | '_ \ / _` |/ _ \ | |_) | | | |_) | '__| | '_ ` _ \ / _ \
| |_| | | | (_| | | | | (_| |  __/ |  __/| | |  __/| |  | | | | | | |  __/
 \___/|_|  \__,_|_| |_|\__, |\___| |_|   |_| |_|   |_|  |_|_| |_| |_|\___|
                       |___/                                              

Welcome to ARMBIAN 5.65 stable Debian GNU/Linux 9 (stretch) 4.14.78-sunxi64
System load:   0.29 0.16 0.06   Up time:       1 min
Memory usage:  3 % of 2002MB    IP:            192.168.100.23
CPU temp:      22°C
Usage of /:    8% of 15G

[ General system configuration (beta): armbian-config ]

New to Armbian? Check the documentation first: https://docs.armbian.com


Thank you for choosing Armbian! Support: www.armbian.com

Creating a new user account. Press <Ctrl-C> to abort

Please provide a username (eg. your forename): 
```

This is a prompt to create a new user, do it so if you like but please notice that you can abort it by hitting Ctrl+C and simply use root account.

Either option (Ctrl+C to exit or create the user) will be kicked out and login prompt will be presented again for you to login.

You are ready to go.

**Note:** Please observe that network manager has already configured your eth0 interface, see the screen snipet above.

### Step 2b: for Raspbian, Start it and change pi password

Log into the system, the default user is this:

```
user: pi
passwd: raspbian
```

Once there change the password of uer pi, just type in the console:

```
sudo passwd pi
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
```

**Warning:** From this point forward users of Raspbian must make a `sudo bash` to switch to root and run all following commands as root.

## Step 3: Get needed files

We will start tinkering with the network now and in the process we may lose connectivity so we will get all the files prior to that, do this on the console:

```
cd /root
git clone http://github.com/stdevPavelmc/armbian-hostpot-bridge.git
```

At the end you will have a folder named armbian-hostpot-bridge in your /root folder with all the data you will need, including this instructions.

### Raspberry Pi install needed files

For the Raspberry Pi you need to install some extra packages, run on the console

```
apt update
apt install hostapd bridge-utils -y
```

## Step 4: Get rid of NetworkManager [Armbian only step]

**This is a armbian only step, if you are using Raspbian, skip to step 5**

By default al interfaces in armbian are handled by NetworkManager, but for the simple bridge to work we don't need it, so we will disable it & remove it from the system. Just type this commands on the console as root:

```
systemctl stop network-manager
systemctl disable network-manager
systemctl disable NetworkManager-wait-online
apt purge network-manager -y
```

With this commands we are disabling & removing the services, at this point our network connection may not work.

## Step 5: Setting up the network bridge [eth0 + wlan0/wlan1]

Just copy the template file we have already downloaded with this command:

```
cd /root/armbian-hostpot-bridge
cp -f interfaces /etc/network/interfaces
```

If you use the default WiFi on the Orange Pi Prime (Or other with onboard WiFi) you are set and there is no need to edit the file, but if you use a USB dongle like in the case of the old Raspberry Pis you have to tweak it, type this on your console to see what is the name of the new USB WiFi dongle:

```
ifconfig -a
```

You will get something like this: 

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether 02:01:55:da:4c:95  txqueuelen 1000  (Ethernet)
        RX packets 1536  bytes 146188 (142.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 283  bytes 29946 (29.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 25  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::e8c:24ff:fe2c:50be  prefixlen 64  scopeid 0x20<link>
        ether 0c:8c:24:2c:50:be  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 442  bytes 57130 (55.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan1: flags=56163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 12af::a105:eff:feaa:5011e  prefixlen 64  scopeid 0x20<link>
        ether 80:e1:00:16:e1:92  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 642  bytes 157130 (155.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

There you can spot a `wlan0` and `wlan1` devices (this is for a Orange Pi Prime + USB WiFi Dongle), the wlan0 is the onboard and the wlan1 is the USB dongle

Now use `nano` to setup your wifi device to wlan1, in the console use:

```
nano /etc/network/interfaces
```

An editor will open with the file content, now sweep trough it and change any wlan0 to wlan1, then save it (F2) (tip, there are 4 places to change, one of them is a comment)

Now if all gone ok you can call the bridge up and test the connection:

```
ifup br0
```

If all worked you enabled the ethernet connectivity again, try to ping a host in your net or the internet to test it.

## Step 6: Configure your hostapd service (Wifi Hotspot)

Just copy the template file we have already downloaded with this command:

```
cd /root/armbian-hostpot-bridge
cp -f hostapd /etc/default/
cp -f hostapd.conf /etc/hostapd/
```

Now is time to edit the WiFi hotspot details, in a console run this to bring up a editor:

```
nano /etc/hostapd/hostapd.conf
```

In there you need to change just three lines, the ones to refers to:

* Wireless device `interface=wlan0`
* Wireless SSID `ssid=OPP-hotspot`
* Wireless Password `wpa_passphrase=123password`

The interface option refers to the device you identified, if it's a SBC with onboard WiFi like the Orange Pi Prime you are set.

The SSID is the name of the wireless network, in this case "OOP-hotspot" and "123passwd" is the password as you may guessed at this time, change at your will.

### Bonus

If yo care of RF spectrum and good link, you can move the WiFi channel usage to a clear one by setting the [channel=6] option to the chosen one.

## You are set!

Power cycle the SBC and enjoy your new bridged hotpot.

## Troubles?

Use the Issues tab of this repo to rise bug reports or ask questions.
