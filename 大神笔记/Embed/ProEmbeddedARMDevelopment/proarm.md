[toc]

## 介绍

本书针对想要进入嵌入式领域的读者。想要了解C语言。不需要汇编。

本书关注最近的ARMv7架构（用于Cortex-A, Cortex-R, Cortex-M）。

开发电脑可以是Linux、Windows或MAC。第一个程序使用免费的ARM模拟器。后面可以使用小型计算机，如树莓派。运行本书例子，需要：

- Linux开发计算机
- Mentor Graphics compiler suite
- Atmel SAM D20 Xplained Pro evaluation board
- Silicon Lab’s STK3200 and STK3800 evaluation boards
- Raspberry Pi

源代码：www.wiley.com/go/profembeddedarmdev

## 1. ARM历史

#### （未）Differences between ARM7TDMI and ARM926EJ-S

### 生产商文档

要了解你使用的处理器，需要了解两件事：

- Processor family
- Architecture version

ARM provides two documents: the processor manual, called the Technical Reference Manual, and the architecture, called the Architecture Reference Manual.

The Architecture Reference Manual lists all the features common to an architecture version, including, but not limited to, assembly instructions, processor modes, memory management systems, and other subsystems. The Technical Reference Manual gives more detailed information about the options and internal information about the current CPU but does not go into architecture details.

For a system on a chip device, the manufacturer normally has extensive documentation. The SoC will be based on an ARM core and ARM architecture, so ARM’s documentation will be necessary, but the manufacturer will also produce a document listing all the electrical characteristics and input/ output information, together with any custom or proprietary peripherals that have been included.

### ARM现在在干嘛？

现在是一个移动设备大量涌现的时代。The Samsung Exynos Octa serves as an example of previously unheard of designs. On one single chip, there are two clusters of processors, a quad-core Cortex-A7 and a quad-core Cortex-A15, for a total of eight cores, and also a graphics processor, as well as numerous peripherals.

CPUs are not the only technology that ARM licenses. One of its concerns when moving away from the 6502 was the chip’s inability to provide good graphics (for the time). Although some embedded systems do not have a screen, others depend heavily on one. A tablet today is only as good as the CPU inside the tablet, but also only as good as the graphics chip. Having a fast CPU isn’t everything; if the web page isn’t displayed fluidly, the system is considered to be useless. ARM develops and licenses Mali graphics processors, achieving more graphics power than some desktop-based graphics cards, while using ultra-low power.

ARM也在新架构ARMv8上花费了大量精力。ARMv8是一大步跨越，因为它向ARM子系统引入了64位能力，同时保持对32位的支持。ARMv8 will open a new market for ARM. Although the low power Cortex-A57 processor could be integrated into a mobile telephone or a tablet, it can also be used on servers, with multiple vendors already working on solutions using ARM technology. A typical server room presents one of the biggest IT costs of any company; the amount of electricity used by servers is phenomenal. Server processors are power hungry, but they also create a lot of heat that has to be evacuated quickly; estimates say that 25 percent of the electricity used is for cooling. Because ARM processors are low-powered and run cool, they are the ideal candidate.

## 2. ARM嵌入式系统

一家公司决定生产一个产品。The specifications are simple: one ARM processor, 2 MB of RAM, and 2 MB of ROM, 16 digital input lines, 8 digital output lines. 使用一张SD卡存储数据，但选用的CPU没有自带SD驱动；必须自制；问题不大；但需要一些额外的开发；用一些数字输入输出完成该问题。自带SD控制器的芯片将更昂贵更大。

注意芯片选定后硬件就固定了。如输出输入线的数量。虽然理论上只用选定芯片的一半，但提放客户增加功能，如WiFi通讯。如果输入输出线不够，就只能重选硬件！

### ARM嵌入式系统定义

#### 片上系统

一些嵌入式系统会尽可能小，只包含绝对必需的组件。另一些设计，使用**片上系统**：一个芯片包含处理器和几乎所有的组件，甚至超出系统需要的组件。过去片上系统非常贵，但现在一些片上系统芯片已经非常便宜了，有时比你采用简单芯片加上额外硬件以及设计印刷线路板的成本更低。但由于SoC芯片晶体管更多，需要需要更大功率。此外，芯片厂商一般会提供软件支持，如操作系统，安装非常快。

Freescale’s iMX 6 series processor contains a DDR controller, four USB2.0 ports, gigabit Ethernet, PCI Express, a GPU, and much more, alongside a quad-core Cortex A9, all on one single chip. The Chinese device-maker Hiapad has created the Hi-802, a tiny complete system only just larger than a USB key, connecting directly to the HDMI port of a television or monitor, which has USB to plug in a keyboard and mouse and integrates Bluetooth, an SD slot, and Wi-Fi. Of course, there are also cheaper versions; a device called the U2 has been selling for as little as $20US, containing a 1.5 GHz Cortex A8, containing again a wireless network card, HDMI interface, USB, ports, and SD-card support.

Manufacturers often have different philosophies concerning SoC chips, and care must be taken when deciding which system to use. Freescale has always been known to make energy-efficient systems, at the cost of slightly degraded processing power. Nvidia, with its Tegra series, have always made multimedia its priority. Samsung makes some of the fastest SoC chips with its Exynos series. Each product range has its advantage, and time must be taken to analyze the best choice possible.

另一个名词是SiP，即System in Package。SiP often includes several chips in one; combining the processor, random access memory, and flash memory.

If your project requires specific hardware and you cannot find a suitable solution on the market, there is always another solution. FPGA SoCs are chips that have an integrated ARM core and enough logic cells to complete your design. The advantage of this sort of platform is the ability to have as much logic as possible on a single chip and entirely adapted to your solution.

#### （未）RISC的优势

### 从哪里开始？

评估板是一个好的起点。调试能力。充足的文档。

ARM provides several evaluation boards. The ARM Versatile Express boards provide excellent training for the Cortex A cores or soft-core Cortex-M. The Versatile range was previously used for Classic processors — from the ARM7TDMI processor up to the ARM1176. The KEIL series are more oriented toward the micro-controller domain and also includes some classic cores.

As well as ARM-based evaluation boards, almost all chip makers have their own evaluation boards. Freescale has some excellent evaluation boards for its iMX line of SoCs, complete with just about every type of connector you can think of. Infineon has a range of clever modular systems, and Texas Instruments has some small-factor systems.

There are multiple ARM-based systems that are not evaluation boards. Moxa creates an impressive amount of industrial systems, and one of my previous clients had a stock of Moxa 7420 systems and builds its software around those systems. The Moxa 7420 is an XScale board running at 533 MHz and has eight serial ports, two Ethernet ports, USB connectivity and CompactFlash storage, along with 128 Mb of RAM and 32 Mb of ROM. With this impressive system, the client reacted quickly to market needs by developing software and a hardware solution for industrial systems on a platform that it knew well.

As explained previously, you need to think carefully about your project and know in advance what is required. Do you need a video controller? How about a SATA controller? An industrial system might not need either but would have a more specific requirement, for example, a CAN bus.

#### 能获得的板子

ARM has an impressive line of evaluation boards, and more information can be found on their website here:
http://www.arm.com/products/tools/development-boards/index.php

Keil, ARM’s tool company, also makes numerous development boards that fit in well with its line of development tools, found here:
http://www.keil.com/boards/

**Arduino Due**

Arduino boards are mostly known for their 8-bit single-board computers. The Arduino family comes with a complete range of shields, ranging from I/O ports to SD-card storage.

尽管整个Arduinos家族都使用8位PIC微控制器，Arduino Due使用一个Cortex-M微控制器。这种板子是用于计算的（即便ARM CPU时钟84 MHz），而是用于基于I/O的电子工程。它有54个数字I/O口，每个都可以编程为输入或输出。有12个模拟输入和2个模拟输出。

You can find Arduinos in a lot of projects based on robotics or sensors, simply because the processor is so heavily based on I/O. It is also hugely popular because it is based on a Cortex-M and is therefore energy-efficient. It is not uncommon to see an Arduino Due run on battery power, and it has been used on mobile robotics, or even autopilot systems for remote controlled planes.

**Raspberry Pi**

They have been used for home automation by using an **I2C** bus, for home media players using the hugely popular XBMC, or for play. Mojang ported the hugely popular game Minecraft to the Raspberry Pi.

Raspberry Pi is a basic computer with all the functionality required for basic systems. It has either 256 Mb or 512 Mb of RAM, one or two USB ports, video output via RCA, HDMI or DSI, audio output, 10/100 Ethernet for the Model B, and an SD slot for the filesystem. There is no on-board storage; the operating system has to be placed onto an SD card. The system is based on an ARM1176JZF-S running at 700MHz with overclocking possibilities.

Raspberry Pi缺乏Arduino的I/O能力，这不是它的兴趣。However, that does not mean that it has no I/O; it does have two I/O ports with GPIO lines. 但处理器上有三分之一的GPIO线是没接得，and some are reserved for the SD card reader and an SPI port. There are numerous projects using the IO lines, but most of these are educational boards mainly focused on turning LEDs on and off.

**Beagleboard**

The Beagleboard is a large system compared to the Raspberry Pi and the Arduino. 它基于Cortex-A8，比Raspberry Pi有更多功能。It has complete video out capabilities, four USB ports, Ethernet, Micro-SD, a camera port, and an expansion port. 有部分I/O能力。The Beagleboard is the computer of the ARM-based development boards. If you are looking for raw processing power above all, this is the system to use.

**Beaglebone**

The Beaglebone is a light version of the Beagleboard. It still has the crunching power and speed of a Beagleboard but is slightly **more** I/O-oriented. 它有更多地输出引脚但没有HDMI头。

Just like the Arduino, the Beaglebone has capes — add-on cards that extend I/O capabilities or add system functionality. There are LCD touch-screen capes, battery capes, and wireless and extended I/O capes. There are also breakout boards, which enable you to create your own circuits.

#### 存在的操作系统

You can concentrate on building your application while letting the operating system handle the hardware technicalities. Memory configuration, networking, and peripheral I/O can be handled directly by the operating system.

**Linux**

Linux has been ported to just about any MMU-enabled processor that exists and has been used on ARM systems for decades. Linux has a huge user base, and the possibility of compiling a home-kernel is a major advantage for embedded systems. You can leave out large sections of the kernel for hardware that you do not need and leave in only the strict minimum. When adding new hardware, there are lots of resources necessary for adding drivers, and it is possible that in the open-source community someone has already developed such a driver.

**VxWorks**

VxWorks是一个实时操作系统，由Wind River设计开发。它有一个多任务的内核，支持多处理器。It is used in a lot of mission-critical systems, where reliability is premium. It powers the on-board computers for the Airbus A400 and powers the radar warning system for the F/A-18 Hornet. VxWorks支持大量空间项目。One of the most famous uses is in the Mars Curiosity rover, where VxWorks was considered to be the only system reliable enough to be placed onto a rover that was sent 350 million miles away, in an environment where nobody can ever perform a hardware update.

**Android**

Android is a Linux-based operating system.

#### 最合适的编译器

Again, several solutions exist. The GNU compiler does an excellent job. ARM也有一个编译器。尽管不是免费得，it has the advantage of having all ARM’s knowledge in a single executable file and can heavily optimize your project.

**GNU Compiler Collection**

The GNU compiler is the entry level **C/C++** compiler. GCC was originally a C compiler, named the GNU C Compiler and was released in 1987. Since then, it has been renamed the GNU Compiler Collection because it now supports many more languages, including different forms of C (**C++**, Objective-C, and *Objective-C++*) as well as other languages (Fortran, Ada, Go, and so on).

GCC naturally supports ARM architectures and has support for different ARM processors and architectures. Instead of compiling for generic ARM, it can be fine-tuned for different architectures, either using or omitting technology as needed. It has full support for the entire ARM Classic collection, the Cortex series, and even the more recent 64-bit Cortex-A53. If you are curious, you can write a small program and see how it would be compiled for an ARM2.

**Sourcery CodeBench**

Sourcery CodeBench is a complete environment with an IDE, debugger, libraries, and support. The Lite Edition, however, has only the command-line tools, namely a custom version of the GCC. Changes are first made in Sourcery CodeBench before being returned to GCC.

Sourcery CodeBench Lite is a complete toolchain, including the compiler, linker, debugger, and just about any tool required to compile C, C++, and assembly.

**ARM Compiler**

Nobody knows ARM processors better than ARM and it has also released the ARM Compiler, the result of more than 20 years of knowledge. This compiler is designed specifically for ARM processors and includes some of the most advanced optimization techniques available. Although this is a professional solution that requires a license, it has exceptional optimization techniques that far surpass GCC.

#### 调试

Debug solutions exist for ARM cores or for ARM-based chips. One of the references in the ARM world is Lauterbach, with its Trace32 solution. Different modules exist enabling assembly-level debugging and displaying internal and external peripherals, hardware breakpoints, and trace solutions. It is possible to debug with only a serial line, and in some cases that is exactly what you have to do. However, more professional solutions can leverage the work load considerably by increasing bandwidth and adding advanced trace functionality.

**Lauterbach Trace32**

Lauterbach produces some extremely good hardware debuggers, notably the PowerDebug and PowerTrace series. These devices can effectively debug in assembly or higher languages such as C and C++. The interface has excellent support for watching variables and displaying memory contents and traces. It has full MMU support, enabling the full display of entries and registers. All the CPU registers and attached peripherals can be controlled directly. Some of the biggest names in the embedded field use Lauterbach devices in their engineering departments.

#### 完整的开发环境

Yes, there are complete solutions that exist, including an IDE, compiler toolchain, and debug tools, all in one package.

ARM DS-5

ARM has its own complete development environment: the ARM Development Studio. At the time of writing, the current version is DS-5.

The DS-5 environment is a professional solution with the Eclipse IDE as a central point and excellent debugging tools, all coupled with the ARM compiler. When coupled with a DSTREAM hardware debugger, it becomes a capable solution with a huge trace buffer that enables long-time traces to be run, even on fast targets. If hardware is not yet available, software emulation is possible with a specialized emulator.

The ARM DS-5 solution also has an option for power monitoring with a device that can read voltage and current, and link capture data with other captures so that you can know when and why a device changes its power settings, and know what portion of code uses the most battery.

The DS-5 solution is a professional solution aimed at engineering teams that work on bleeding- edge applications that must have the upper-hand in code efficiency. Solutions of this quality are not cheap, but ARM has managed to make the price extremely reasonable, and everything you need is contained in a single package.
You can find more information at http://www.arm.com/products/tools/software-tools/ds-5/index.php.

ARM DS-5 Community Edition

Compared to the professional DS-5 solution, this version is also managed by ARM but has some limitations and does not come with the ARM compiler. It does, however, come with function profiling, process tracing, and a limited set of performance counters. Debugging is done with software because there is no hardware debugger included, but GDB is a powerful tool, and the DS-5 CE completes that beautifully.

DS-5 CE is maintained by ARM and has the same quality as the professional build, but with open source tools and limited functionality. It is a great platform to begin working on to get used to the DS-5 environment The ARM forums are the place to look for information and to ask questions.

#### 其他

Modern systems can communicate either by Ethernet or USB, but embedded systems almost always have a serial port. Using a serial port is much easier than using any other device such as I2C or CAN. With a serial device, you put bytes into a specific register, and that is just about it. For this reason, and because there is little software needed to make it function, almost all embedded devices have a serial terminal.













