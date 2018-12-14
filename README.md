# Armbian as a wireless router connected to your local LAN using a Orange Pri Prime SBC

Main goal is to create detailed instructions to make a armbian wireless router with full security that allows you to use your home lan to route traffic.

All this over vanilla armbian and as easy as possible.

## Requirements

You will need the following to complete this guide:

* Orange Pi Prime SBC
* uSD card, at least 8 GB and class 10 recommended.
* USB keyboard
* HDMI cable
* HDMI capable monitor
* Power adapter for the Orange Pi Prime SBC  (5.0 V @ 2A or more)
* A free ethernet port on your home router with internet access
* [Optional] if your lan has no DHCP you will need to know your network data (IP, mask, gateway)

Also I will assume that your local network has a DHCP server, this http server will server wifi users as well; we will bridge the LAN and WIFI networks.

## Step 1: Get armbian and flash it

Get armbian for Orange Pi Prime SBC, you can download the image from the official site [here](https://www.armbian.com/orange-pi-prime/) pick the non desktop one as we will require not UI.

Once you have it un-compress the file and flash it to a uSD card, you can find a lot of guides about how to do it over the internet.

## Step 2: Start it and create the user.

Insert the uSD card into the Orange Pi Prime SBC, connect the ethernet cable from your local lan, the monitor and the usb Keyboard; at the end connect the power cable.

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

## Step 3: Get needed files

We will start tinkering with the network now, and in the process wil will lose connectivity, so we will get all the files, do this on the console.

```
cd root
git clone http://github.com/stdevPavelmc/armbian-hostpot-bridge.git
```

At the end you will have a folder named armbian-hostpot-bridge in your /root folder with all the data you will need, including this instructions.

## Step 4: Get rid of NetworkManager

By default al interfaces in armbian are handled by NetworkManager, but for the simple bridge to work we don't need it, so we will disable it & remove it from the system. Just type this commands on the console as root:

```
systemctl stop network-manager
systemctl disable network-manager
systemctl disable NetworkManager-wait-online
apt purge network-manager -y
```

By this we are disabling & removing the services, at this point out network connecction will not work.

## Step 5: Setting up the network bridge [eth0 wlan0/wlan1]

Just copy the template file we have already downloaded with this command:

```
cd /root/armbian-hostpot-bridge
cp -f interfaces /etc/network/interfaces
```

If you use the default WiFi on the Orange Pi Prime you are set and there is no need to edit the file, but if you use a USB dongle you have to tweak it, type this on your console to see what is the name of the new USB WiFi dongle:

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

There you can spot a `wlan0` and `wlan1` devices, the wlan0 is the onboard and the wlan1 is the USB dongle

Now use `nano` to setup your wifi device to wlan1, in the console use:

```
nano /etc/network/interfaces
```

An editor will open with the file content, now sweep trough it and change any wlan0 to wlan1, then save it (F2) (tip, there are 4 places to change, one of them is a comment)

Now if all gone ok you can call the bridge up and test the connection:

```
ifup br0
```

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

In there you need to change just two lines, the ones to refers to:

* Wireless SSID [ssid='OPP-hotspot']
* Wireless Password [wpa_passphrase='123password']

In this case "OOP-hotspot" is the WiFi network name and '123passwd' is the password, change at your will but take care of keep them between the single quotes to avoid interpretation of spaces or special chars

### Bonus

If yo care of RF spectrum polution and good link, you can move the WiFi channel usage to a clear one by setting the [channel=6] option to the chosen one.

## You are set!

Power cycle the Orange Pi Prime and enjoy your new bridged hotpot.
