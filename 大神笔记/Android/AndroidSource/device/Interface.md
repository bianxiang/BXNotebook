[toc]

## Android Interfaces

Android gives you the freedom to implement your own device specifications and drivers. The hardware abstraction layer (HAL) provides a standard method for creating software hooks between the Android platform stack and your hardware.

Android系统架构
![](img/ape_fwk_all.png)

**Binder IPC**

The Binder Inter-Process Communication (IPC) mechanism allows the application framework to cross process boundaries and call into the Android system services code. This enables high level framework APIs to interact with Android system services.

**系统服务**

Functionality exposed by application framework APIs communicates with system services to access the underlying hardware. Services are modular, focused components such as Window Manager, Search Service, or Notification Manager. Android包括两种服务： system (services such as Window Manager and Notification Manager) 和 media (services involved in playing and recording media)。

**硬件抽象出（HAL）**

The HAL is a standard interface that allows the Android system to call into the device driver layer while being agnostic about the lower-level implementations of drivers and hardware.

You must implement the corresponding HAL (and driver) for the specific hardware your product provides. HAL implementations are typically built into shared library modules (.so files). Android does not mandate a standard interaction between your HAL implementation and your device drivers, so you have free reign to do what is best for your situation. However, to enable the Android system to correctly interact with your hardware, you must abide by the contract defined in each hardware-specific HAL interface.

**Linux内核**

Developing your device drivers is similar to developing a typical Linux device driver. Android uses a version of the Linux kernel with a few special additions such as wake locks (a memory management system that is more agressive in preserving memory), the Binder IPC driver, and other features important for a mobile embedded platform. These additions are primarily for system functionality and do not affect driver development.

You can use any version of the kernel as long as it supports the required features (such as the binder driver). However, we recommend using the latest version of the Android kernel. For details on the latest Android kernel, see Building Kernels.


