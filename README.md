# Technexion implementation of camera driver for TI EVK board

[![TEVS_Banner](https://github.com/user-attachments/assets/02219e99-b031-4a44-84c3-75277ed1a4ec)](https://www.technexion.com/products/embedded-vision/)


[![Producer: TechNexion](https://img.shields.io/badge/Producer-Technexion-blue.svg)](https://www.technexion.com)
[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
<br/><br/>

This repository contains camera driver and device tree blobs (DTBs) for [TechNexion Embedded Vision Solutions](https://www.technexion.com/products/embedded-vision/) on Texas Instruments EVK board.

Refer to the [TEVS Camera Usage Guide for Texas Instruments EVK](https://tn-docusaurus.vercel.app/docs/embedded-vision/tevs/usage-guides/ti/), this guide offers a comprehensive walkthrough for utilizing TechNexion's TEVS camera modules on the Ti EVK board. It covers essential steps from hardware setup, including connecting the adapter board and camera cable to the MIPI CSI port, to preparing and flashing the Yocto demo image. <br/><br/>
The guide also details the process of configuring the media controller, starting the camera video stream via GStreamer, and troubleshooting common issues. Additionally, it provides instructions for building a Yocto-based Linux image tailored for camera modules, ensuring developers have all the necessary tools and knowledge to accelerate their embedded vision projects. This resource is invaluable for embedded system developers looking to leverage high-performance, industrial-grade camera solutions for applications in industrial automation, prototyping, and edge AI.

## Driver Installation Instructions

1. ### **Download the camera driver and device tree blobs.**
    ```shell
    $ git clone git@github.com:TechNexion-Vision/tn-ti_6.1.46_09.01.00.006.git
    $ cd ti_evk_camera/
    ~/ti_evk_camera$ git checkout tn-ti_6.1.46_09.01.00.006
    ```
2. ### **Copy to your kernel source code.**
    ```shell
    ~/ti_evk_camera$ mv -r driver/media/i2c/tevs/ <fetch_kernel_folder>/driver/media/i2c/
    ~/ti_evk_camera$ mv arch/arm64/boot/dts/ti/k3-j721e-sk-tevs.dtso <fetch_kernel_folder>/arch/arm64/boot/dts/ti/
    ```
3. ### **Modify makefile to add driver**
    ```shell
    $ cd <fetch_kernel_folder>/driver/media/i2c/
    ~/<fetch_kernel_folder>/driver/media/i2c/$ vi Makefile
    ```
    Add this line in Makefile.
    ```shell
    obj-$(CONFIG_VIDEO_TEVS) += tevs/
    ```
    Modify Kconfig to add camera config
    ```shell
    ~/<fetch_kernel_folder>/driver/media/i2c/$ vi Kconfig
    ```
    Add this part under "Camera sensor devices" menu in Kconfig.
    ```
    config VIDEO_TEVS
      tristate "TechNexion TEVS sensor support"
      depends on OF
      depends on GPIOLIB && VIDEO_V4L2 && I2C && VIDEO_V4L2_SUBDEV_API
      depends on MEDIA_CAMERA_SUPPORT
      default y
      select V4L2_FWNODE
      help
        This is a Video4Linux2 sensor driver for the TechNexion
        TEVS camera sensor with a MIPI CSI-2 interface.
    ```

4. ### **Modify makefile to add device tree.**

    ```shell
    $ cd <fetch_kernel_folder>/arch/arm64/boot/dts/ti/
    ~/<fetch_kernel_folder>/arch/arm64/boot/dts/ti/$ vi Makefile
    ```
    Add this line in Makefile.
    ```
    #For SK-AM62X
    dtb-$(CONFIG_ARCH_K3) += k3-am625-sk-csi2-tevs.dtbo
    #For SK-AM62A
    dtb-$(CONFIG_ARCH_K3) += k3-am62a7-sk-csi2-tevs.dtbo
    #For SK-TDA4VM
    dtb-$(CONFIG_ARCH_K3) += k3-j721e-sk-tevs.dtbo
    ```

5. ### **Compile the kernel & module driver**

    Finally you can start compile your new Image files, then copy and replace the Image files and add camera dtb overlay file in the SD card.<br/>

> [!NOTE]
> You need specifying the SD card image of dtb overlay file in `/run/media/BOOT-mmcblk1p1/uEnv.txt` as below,<br/>
> ```
> #For SK-AM62X
>   name_overlays= k3-am625-sk-csi2-tevs.dtbo
> #For SK-AM62A
>   name_overlays= k3-am62a7-sk-csi2-tevs.dtbo
> #For SK-TDA4VM
>   name_overlays=ti/k3-j721e-edgeai-apps.dtbo ti/k3-j721e-sk-tevs.dtbo
> ```

  <br/>
  <br/>
  <br/>
