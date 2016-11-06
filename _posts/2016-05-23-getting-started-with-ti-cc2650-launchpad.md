---
layout: post
title:  "Getting Started with TI CC2650 LaunchPad and Contiki"
date:   2016-05-23
comments: true
---

I just bought two [TI CC2650 LaunchPads](http://www.ti.com/tool/launchxl-cc2650) from [Mouser](http://www.mouser.ie/ProductDetail/Texas-Instruments/LAUNCHXL-CC2650/?qs=%2fha2pyFadug%2fd2ESpkXAEG2IdWRgnoCCyoV23EIP4Ml1o1iQsexXhHkiZkooXnRl), which feature the recently released [TI CC2650 System-on-a-Chip (SoC)](http://www.ti.com/product/CC2650), and I thought I could write a simple tutorial to help people get started with the LaunchPad and [Contiki OS](http://www.contiki-os.org) on Mac OS X. 

The CC2650 SoC integrates an [ARM Cortex-M3](http://www.arm.com/products/processors/cortex-m/cortex-m3.php) microcontroller together with a [Bluetooth Low Energy (BLE)](https://www.bluetooth.com/what-is-bluetooth-technology/bluetooth-technology-basics/low-energy) and [IEEE 802.15.4](https://standards.ieee.org/about/get/802/802.15.html) radio chip. This SoC brings new opportunities for wireless devices and research. For instance, [Thingsquare](http://www.thingsquare.com) uses BLE for [proximity control](http://www.thingsquare.com/blog/articles/proximity-control/) using the [SensorTag](http://www.ti.com/ww/en/wireless_connectivity/sensortag2015/?INTC=SensorTag&HQS=sensortag), another TI CC2650-based evaluation board. Other researchers are studying how to run RPL over BLE, see this [demo](http://netlab.snu.ac.kr/~hskim/paper/Conf_2015_SenSys(2).pdf) and this [paper](http://netlab.snu.ac.kr/~hskim/paper/Conf_2016_SECON.pdf). 

The following picture obtained from the [TI CC2650 LaunchPad Quick Start Guide](http://www.ti.com/lit/ml/swru451/swru451.pdf) details some of the LaunchPad features.

![TI CC2650 LaunchPad](/img/cc2650_lp_explained.png)

# Requirements

Everything that you need to get started with the TI CC2650 LaunchPad and Contiki is enumerated in the Contiki TI CC2650 platform's [README](https://github.com/contiki-os/contiki/tree/master/platform/srf06-cc26xx). Here, you can find the commands to install all these requirements on Mac OS X.

First, clone the [Contiki GitHub repository](https://github.com/contiki-os/contiki) and get its submodules. Specifically, the [CC26XXware](https://github.com/g-oikonomou/cc26xxware) and the [CC2538-bsl script](https://github.com/JelmerT/cc2538-bsl) submodules are required.

{% highlight bash %}
$ git clone https://github.com/contiki-os/contiki
$ cd contiki
$ git submodule update --init
{% endhighlight %}

The previous commands should download the latest Contiki and its submodules. If you already have a previous clone of Contiki, you can update it with `git pull`. If you have a Contiki fork that is not up-to-date you can run the following commands:

{% highlight bash %}
$ git checkout master
$ git fetch upstream
$ git merge upstream/master
$ git push origin master
{% endhighlight %}

Being `upstream` the original project repository. If you do not have any `upstream` added, you can add it like this: `git remote add upstream https://github.com/contiki-os/contiki`. 

Then, if the submodules in your fork or clone are not up-to-date, you can update them with the following command:

{% highlight bash %}
$ git submodule foreach git pull origin master
{% endhighlight %}

By now, you should have the latest Contiki, CC26XXware, and the CC2538-bsl script that you need to upload an image over the serial port to the LaunchPad. However, you may still not have a toolchain to build the firmware. To this end, you need the [GNU Tools for ARM Embedded Processors](https://launchpad.net/gcc-arm-embedded). On Mac OS X to install these tools, you can use [Homebrew](http://brew.sh) as follows:

{% highlight bash %}
$ brew cask install gcc-arm-embedded
{% endhighlight %}

At the time of writing, this installs version 5.2.1.

{% highlight bash %}
$ arm-none-eabi-gcc --version
arm-none-eabi-gcc (GNU Tools for ARM Embedded Processors) 5.2.1 20151202 (release) [ARM/embedded-5-branch revision 231848]
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
{% endhighlight %}

Finally, you will also need to install [SRecord](http://srecord.sourceforge.net). You can also use Homebrew for this purpose:

{% highlight bash %}
$ brew install srecord
{% endhighlight %}

# Flashing the LaunchPad with TI SmartRF Flash Programmer 

Most likely, your LaunchPad comes pre-programmed with factory firmware that does not have the [ROM bootloader]((http://www.ti.com/lit/an/swra466a/swra466a.pdf)) enabled. As a consequence, before you can use the [CC2538-bsl script](https://github.com/JelmerT/cc2538-bsl) to programme the LaunchPad over serial, you will need to flash first the CC2650 with a new image that enables the [device bootloader](http://www.ti.com/lit/an/swra466a/swra466a.pdf). To do that, you will need to modify the [`ccfg.c` file](https://github.com/g-oikonomou/cc26xxware/blob/0270b50ac750f8f3348a98f900a470e7a65ffce8/startup_files/ccfg.c) that you can find in:

`contiki/cpu/cc26xx-cc13xx/lib/cc26xxware/startup_files/`

Specifically, you need to enable the ROM bootloader and its backdoor, define the pin state to activate the bootloader backdoor, and the DIO number of the backdoor. You can do that by looking for the bootloader definitions in the `ccfg.c` file and modifying them as follows:

{% highlight c %}
#define SET_CCFG_BL_CONFIG_BOOTLOADER_ENABLE            0xC5       // Enable ROM boot loader
#define SET_CCFG_BL_CONFIG_BL_LEVEL                  	0x0        // Active low to open boot loader backdoor
#define SET_CCFG_BL_CONFIG_BL_PIN_NUMBER                0x0D       // DIO number for boot loader backdoor
#define SET_CCFG_BL_CONFIG_BL_ENABLE                 	0xC5       // Enabled boot loader backdoor
{% endhighlight %}

With these modifications you are setting the button 1 (`DIO13`) to enable the bootloader. If you prefer for any reason to use button 2 for this purpose, simply set:

{% highlight c %}
#define SET_CCFG_BL_CONFIG_BL_PIN_NUMBER                0x0E       // DIO number for boot loader backdoor
{% endhighlight %}

Once, you have done these modifications and you have all the requirements, you can compile a new image to flash the TI CC2650 LaunchPad. For simplicity, I will use in this post the `cc26xx-demo.c` example in Contiki. You can compile the image for the LaunchPad as follows:

{% highlight bash %}
$ cd contiki/examples/cc26xx/
$ make BOARD=launchpad/cc2650 cc26xx-demo
{% endhighlight %}

After this, you should be able to see the next files:

{% highlight bash %}
$ ls cc26xx-demo.*
cc26xx-demo.bin			cc26xx-demo.c			cc26xx-demo.elf			cc26xx-demo.hex			cc26xx-demo.srf06-cc26xx
{% endhighlight %}

Basically, the `cc26xx-demo.elf` or `cc26xx-demo.hex` is the image that you need to flash the TI CC2650 LaunchPad. To do so, you will need the [TI SmartRF Flash Programmer](http://www.ti.com/tool/flash-programmer), which you can [download for free from TI](http://www.ti.com/tool/flash-programmer). Download the second version (v2). You will need to first sign up and then fill a form to download the software. Unfortunately, TI SmartRF Flash Programmer is only available for Windows, so you will need to either use a Windows machine or a virtual machine with [VMware Fusion](https://www.vmware.com/products/fusion) or [Parallels](http://www.parallels.com/eu/). A priori, you could also use [VirtualBox](https://www.virtualbox.org). However, with VirtualBox I had a problem when upgrading the firmware in the XDS110 MCU, which is most likely the first thing you will have to do when using for the first time your LaunchPad together with SmartRF Flash Programmer.

Once you have TI SmartRF Flash Programmer up and running, connect your LaunchPad to your Windows (virtual) machine and you will see a device appearing in the left pannel of the application. Click on it and update if required the XDS110 firmware. Then, connect to the LaunchPad using the same pannel. Once the connection is successful, select the image to upload to the LaunchPad, i.e., either the `cc26xx-demo.elf` or the `cc26xx-demo.hex` files. Then, mark the three checkboxes available in the application, i.e., erase, program, and verify. Click play and wait to get the green bar reporting success. Congratulations you have programmed your LaunchPad for the first time!

# Flashing the LaunchPad with the CC2538-bsl script

Now that your LaunchPad is running a firmware that has the the ROM bootloader enabled, you should be able to programme the LaunchPad over serial using the [CC2538-bsl script](https://github.com/JelmerT/cc2538-bsl) on Mac OS X. Note that you should have the CC2538-bsl script dependencies installed. 

First, you need to manually enable the bootloader backdoor. To do so, connect your LaunchPad to your Mac OS computer, press and hold Button 1, reset the LaunchPad, and then release Button 1. Now, you can directly programme the LaunchPad as follows:

{% highlight bash %}
$ make BOARD=launchpad/cc2650 cc26xx-demo.upload
{% endhighlight %}

If by any chance, you are getting an error such as `ERROR: Timeout waiting for ACK/NACK after 'Synch (0x55 0x55)'`, you may need to reduce the baudrate by modifying the `BSL_FLAGS` of the CC2538-bsl script either in `platform/srf06-cc26xx/Makefile.srf06-cc26xx` or in `platform/srf06-cc26xx/launchpad/Makefile.launchpad` to:

{% highlight bash %}
BSL_FLAGS += -e -w -v -b 115200
{% endhighlight %}

# Connecting via serial port to the LaunchPad

To connect via serial port to your TI CC2650 LaunchPad, first you need to see the serial port the LaunchPad is connected to. To do that just run:

{% highlight bash %}
$ ls /dev/tty.usbmodem*
/dev/tty.usbmodemL1002961	/dev/tty.usbmodemL1002964
{% endhighlight %}

In my particular case, the LaunchPad is connected to the `/dev/tty.usbmodemL1002961` serial port. Then, I can use the `serialdump-macos` tool of Contiki to connect serially to the LaunchPad with the following command:

{% highlight bash %}
$ make login PORT=/dev/tty.usbmodemL1002961
{% endhighlight %}

Running that command you should see something like:

{% highlight bash %}
$ make login PORT=/dev/tty.usbmodemL1002961
using saved target 'srf06-cc26xx'
../../tools/sky/serialdump-macos -b115200 /dev/tty.usbmodemL1002961
connecting to /dev/tty.usbmodemL1002961 (115200) [OK]
-----------------------------------------
Bat: Temp=22 C
Bat: Volt=3339 mV
Starting Contiki-3.x-2405-g562a33a
With DriverLib v0.44336
TI CC2650 LaunchPad
 Net: sicslowpan
 MAC: CSMA
 RDC: ContikiMAC, Channel Check Interval: 16 ticks
 RF: Channel 25
CC26XX demo
-----------------------------------------
Bat: Temp=26 C
Bat: Volt=3292 mV
Left: Pin 0, press duration 52 clock ticks
Left: Pin 0, press duration 121 clock ticks
Left: Pin 0, press duration 277 clock ticks
Long button press!
{% endhighlight %}

I hope this helps you! 
