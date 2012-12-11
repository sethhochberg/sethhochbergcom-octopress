---
layout: post
title: USB RNDIS Modem as WAN on Tomato Firmware
comments: false
categories: linux
---

I recently ran into a situation where it was desirable for me to be able to plug my Clear Clearstick Atlas (a plug-and-play USB WiMax modem) into the USB port on my router and use it as the internet connection for all the devices on my network. Below is how I accomplished this task using my Netgear WNR3500L router, running the .... build of the awesome Tomato router firmware. 

First, I set up the Clearstick in it's NAT operation mode, rather than IP passthrough. This let me use the Clearstick as a traditional modem, where the modem assigns an IP address to the router via DHCP. In the Clearstick's configuration: 

Modem IP: 192.168.1.10 (for the Clearstick)  
Host IP: 192.168.1.40 (for the router)

A few kernel modules are required to get the Clearstick recognized by the router. You'll need the following files: 

mii.ko  
usbnet.ko  
cdc_ether.ko  
rndis_host.ko  

You can get these from the "extras-mips" tarballs from whatever the latest build available is here: 
[Shibby's Tomato Site](http://tomato.groov.pl/download/K26/)

The best place to put these files is in the router's JFFS partition, so they're available to you even after reboots of the hardware. Google around for a tutorial on setting that up, there are plenty available. 

Once you've got the kernel modules onto the router, insert them in the order they are listed above like so: 

``` bash
$ insmod mii.ko  
$ insmod usbnet.ko
```

...etc...

After that is done, check to see that all the right modules are loaded: 

``` bash
$ lsmod
```

If everything looks good there, plug in the Clearstick, wait for it to indicate that it has a signal, and then check to make sure you have a new network interface available. Make note of what the network interface is (for me, the new CDC_Ether device was eth2)

``` bash
$ dmesg
```

When we've got the interface name, now we just need to bring the interface up, have the router request an IP via DHCP from the modem, add the modem to the default bridge (so our other interfaces can talk to it), and make sure we route all outbound traffic through the modem. Use these commands. Substitute your interface name for eth2 and the modem's IP for 192.168.1.10. 

``` bash
$ ifconfig eth2 up  
$ udhcpc -i eth2  
$ brctl addif br0 eth2  
$ route add default gw 192.168.1.10
```

In the router's configuration menu, set the DNS servers to 8.8.8.8 and 8.8.4.4 - this isn't really required, but I'd rather use Google's DNS than Clearwire's.  