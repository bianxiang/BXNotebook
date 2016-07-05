[toc]

## 1. Embedded Linux for Makers

The BeagleBone is an embedded Linux development board that’s aimed at hackers and tinkerers. It’s a smaller, more barebone version of their BeagleBoard. Both are open source hardware and use **Texas Instruments**’ processors with an **ARM Cortex-A** series core, which are designed for low-power mobile devices.

### （未）Why Use BeagleBone?

与树莓派对比：

There’s a lot of buzz around Raspberry Pi, and while it’s quite similar to the BeagleBone, there are certainly a few differences. For one, the Raspberry Pi is meant as a low-cost computer to encourage the younger generation to learn about how computers work and how to program them. Because of that, the hardware, software, and documentation are geared towards that objective. On the other hand, the BeagleBone is aimed more broadly at people interested in embedded Linux development boards and therefore has more options for connecting hardware and has a more powerful processor.

## 2. 基础和准备

There are two versions of the board: the original BeagleBone and the newer BeagleBone Black.

It’s quite easy to tell the boards apart. The original BeagleBone is mostly white with black lettering and the BeagleBone Black is mostly black with white lettering. The main improvements with the BeagleBone Black are a faster processor, more memory, on-board storage, and on-board video output. Not only that, but the BeagleBone Black is half the price of the original BeagleBone.

### 组成

1. 处理器。原来的BeagleBone的处理器能力与**iPhone 4**相当。If you like to hear the numbers, it’s a 720MHz ARM Cortex-A8 equipped with 256 MB of DDR2 RAM. If you have a BeagleBone Black, it’ll be a 1GHz chip with 512MB of DDR3 RAM.
2. 电源连接。需要5、500mA直流。Right nearby the jack is a small power a protection chip in case you accidentally provide over five and up to 12 volts.
3. 以太网口。
4. Reset按钮。Press this button to reboot the board. 与电脑一样，最好通过系统重启，而不是这个按钮。
5. USB Host Port. 可以接键盘、WiFi适配器等。
6. LEDs. Next to the power connector, you have an LED to indicate when power is applied to the board. There are also four LEDs next to the reset button that can be programmed by you with software. By default, LED 0 will show a “heartbeat” when the system is running. LED 1 will blink when the MicroSD card is being accessed. LED 2 will blink when the CPU is active, and LED 3 will blink when the on-board flash memory is being accessed (on the BeagleBone Black).
7. Expansion Headers. These two expansion headers, labeled P8 and P9, allow you to integrate your BeagleBone into electronics projects. The pins can be configured for a number of different functions, which we’ll dive into in Chapter 4.
8. Mini USB Port. This USB port allows your BeagleBone to act as a device when you connect it to your computer. Your computer will not only provide power to the board over USB, but it also acts as a means of communicating with it. You can also access BeagleBone reference information stored on-board; it will simply appear as a storage device when you plug it in to your computer. If you choose to power your board through this port, the processing speed will be reduced to decrease its power consumption.
9. MicroSD Card Slot. BeagleBone没有硬盘，使用一个MicroSD卡存储操作系统、程序、数据。On a BeagleBone Black, the operating system is stored on the onboard flash memory (see below) and can be updated using the MicroSD card slot.
10. Micro HDMI Port (BeagleBone Black only). To connect the BeagleBone Black to a monitor or television, use the Micro HDMI port.
11. Serial Header (BeagleBone Black only). While both the original BeagleBone and the BeagleBone Black have serial outputs for accessing the terminal, only the BeagleBone Black breaks out one of the serial ports in its own header. The layout on this header makes it easy to connect an FTDI TTL-232 cable or breakout board so that you can use the text based terminal via USB.
12. On Board Flash Memory (BeagleBone Black only). The BeagleBone Black sports on-board flash memory and can therefore be booted without a MicroSD card inserted. In the technical manuals, this memory is referred to as the **eMMC**.
13. Boot Switch (BeagleBone Black only). Holding down the boot switch when you power on your BeagleBone Black instructs the hardware to boot from the MicroSD card instead of the on-board flash memory.

### 操作系统

While there are many different flavors, or distributions of Linux out there, BeagleBoard.org offers a distribu- tion called Ångström that’s tailored for the board.

A factory-fresh BeagleBone Black will have Ångström preloaded onto the on-board flash memory, or eMMC. If you have an original BeagleBone, it will come with a microSD card with Ångström on it. Since development on this distribution happens rapidly, it’s a good idea to stay up-to-date to the latest version. Appendix A walks you through how to create an up-to-date MicroSD card.

### 连接到BeagleBone

#### Connecting via USB and Installing Drivers


1. If you have an original BeagleBone, be sure that a MicroSD card with the latest version of the BeagleBone Ångström image is inserted into the slot.
2. Connect the BeagleBone to your computer via a USB A to mini-B cable
3. After about 20 seconds, a drive called BEAGLEBONE should appear in your filesystem’s disk volume list. Open that drive and double click on the START HTML document (START.htm) to open it up in your default web browser.
4. Follow the instructions in the “Install Drivers” section on that page for your operating system.
5. In your web browser, go to http://192.168.7.2/ to open Bone 101. This page is being served by your BeagleBone and has a lot of information about the board, including some interactive examples of Bonescript, a JavaScript library written for the BeagleBone.

#### Connecting via SSH over USB

1. Open your terminal and connect to the BeagleBone: type `ssh root@192.168.7.2`
3. There’s no password set by default, so if it prompts you, just hit enter.
4. You know you’re connected when you see the prompt: `root@beaglebone:~#`

#### Connecting via SSH over Ethernet

3. Connect to it via SSH: type `ssh root@beaglebone.local`
5. There’s no password set by default, so if it prompts you, just hit enter.

> If you’re on Windows and the host name **beaglebone.local** does not work, you may need to download Bonjour Print Services for Windows. You can also use the IP address of the board instead. Find it by logging into your router and looking for “beaglebone” on the DHCP clients list.

#### 使用键盘、显示器、鼠标

If you have a BeagleBone Black, you can use it directly by connecting an HDMI monitor, keyboard, and mouse. Since there’s only one USB host port on the BeagleBone, you’ll need to use a USB hub to connect both the keyboard and mouse. When you boot up the BeagleBone Black, you’ll be presented with the GNOME desktop environment.

#### （未）Connecting via Serial over USB

### （未）3. Getting Around with Linux

### 4. 数字电路初步

























