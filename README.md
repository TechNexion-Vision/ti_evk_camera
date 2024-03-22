# TI TDA4VM Camera Driver

[![p4](https://github.com/TechNexion-Vision/ti_evk_camera/assets/57210123/ca72eb4a-e9e2-4307-8e3f-3ee8b89f75b7)](https://www.technexion.com/products/embedded-vision/)


[![Producer: Technexion](https://img.shields.io/badge/Producer-Technexion-blue.svg)](https://www.technexion.com)
[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
<br/><br/>

## Overview
Driver has been developed by TechNexion as an initiative in order to bootup Technexion MIPI-CSI2 cameras for [ROCESSOR-SDK-LINUX-SK-TDA4VM](https://www.ti.com/tool/PROCESSOR-SDK-J721E)<br/>

## Prerequisites
This release have been tested with:
+ **TI TDA4VM Edge AI base Kit**： [J721EXSKG01EVM ](https://www.ti.com/tool/SK-TDA4VM?keyMatch=&tisearch=search-everything&usecase=hardware#order-start-development)<br/>
+ **camera + RPI15 adapter board**： [Camera with RPI15](https://www.technexion.com/?s=RPI15&post_type=product)<br/>
The SK-TDA4VM board provides 2 RPi headers for camera connection.<br/>
  ![S__60_720p](https://github.com/TechNexion-Vision/ti_evk_camera/assets/57210123/7c4a5deb-a138-456f-9716-cf6ed082f93a)<br/>
Fig. 1.1 TechNexion camera module connection with TDA4VM Starter Kit for Edge AI
<br/><br/>
+ **Yocto prebuild image** <br/>
  [tisdk-edgeai-image-j721e-evm.wic.xz](https://dr-download.ti.com/software-development/software-development-kit-sdk/MD-4K6R4tqhZI/09.01.00.06/tisdk-edgeai-image-j721e-evm.wic.xz)
      (Version: 09.01.00.06 Release date: 08 Dec 2023)<br/>
  `xz -d tisdk-edgeai-image-j721e-evm.wic.xz`
  
+ **Micro-SD Card** <br/>
  Micro-SD Card capacity must be higher 16GB and then flash the image.<br/>
  `sudo dd of=/dev/<sd card> if=tisdk-edgeai-image-j721e-evm.wic bs=1M status=progress`
<br/><br/>
## Driver Installation instructions

**1. Download the camera driver and device tree blobs.**
  ```
  $ git clone git@github.com:TechNexion-Vision/tn-ti_6.1.46_09.01.00.006.git
  $ cd ti_evk_camera/
  ~/ti_evk_camera$ git checkout tn-ti_6.1.46_09.01.00.006
  ```
**2.  Copy to your kernel source code.**
  ```
  ~/ti_evk_camera$ mv -r driver/media/i2c/tevs/ <fatch_kernel_folder>/driver/media/i2c/
  ~/ti_evk_camera$ mv arch/arm64/boot/dts/ti/k3-j721e-sk-tevs.dtso <fatch_kernel_folder>/arch/arm64/boot/dts/ti/
  ```
  **3.  Modify makefile to add driver**
  ```
  $ cd <fatch_kernel_folder>/driver/media/i2c/
  ~/<fatch_kernel_folder>/driver/media/i2c/$ vi Makefile
  ```
  Add this line in Makefile.
  ```
  obj-$(CONFIG_VIDEO_TEVS) += tevs/
  ```
  Modify Kconfig to add camera config
  ```
  ~/<fatch_kernel_folder>/driver/media/i2c/$ vi Kconfig
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

  **4.  Modify makefile to add device tree.**

  ```
  $ cd <fatch_kernel_folder>/arch/arm64/boot/dts/ti/
  ~/<fatch_kernel_folder>/arch/arm64/boot/dts/ti/$ vi Makefile
  ```
  Add this line in Makefile.
  ```
  dtb-$(CONFIG_ARCH_K3) += k3-j721e-sk-tevs.dtbo
  ```
<br/>

  **5.  Compile the kernel & module driver**
  
  Finally you can start compile your new Image files, then copy and replace the Image files and add camera dtb overlay file in the SD card.<br/>
<br/>
  > [!NOTE]
  > you need specifying the dtb overlay file in `/run/media/BOOT-mmcblk1p1/uEnv.txt` as below,<br/>
  > `name_overlays=ti/k3-j721e-edgeai-apps.dtbo ti/k3-j721e-sk-tevs.dtbo`

## Instructions for testing camera
  
1. Connect debug console cable to NXP 8MPLUSLPD4-EVK.
2. Power on the board.
   The display will shown an Edge AI demo wallpaper as below.<br/>
   <img src="https://github.com/TechNexion-Vision/ti_evk_camera/assets/57210123/8146a947-d875-4c9f-96de-4df19003d1db" width="400" height="250"><br/>
   Fig. 1.2 TDA4VM Starter Kit wallpaper upon boot<br/>
  > [!NOTE]
  > You can stop and disable this by using `systemctl stop edgeai-init.service` and `systemctl disable edgeai-init.service`.<br/>
  > You won't see it the next time reboot.

  Checking whether camera have been link via v4l2 control command :
  ```
  $ v4l2-ctl --list-device
  ```
  Get the device number of camera capture device.
  ```
  j721e-csi2rx (platform:4510000.ticsi2rx):
        /dev/video2
  ```
  or
  ```
  j721e-csi2rx (platform:4510000.ticsi2rx):
        /dev/video2
  ```
  ![image](https://github.com/TechNexion-Vision/ti_evk_camera/assets/57210123/a36e193c-93cc-4a9e-8ea5-8b4a64146465)<br/>
  Fig. 1.3 Camera connected on CMA2 port<br/>
  <br/>
  You also can check boot log to find camera is probing success<br/>
  ![image](https://github.com/TechNexion-Vision/ti_evk_camera/assets/57210123/2a4d31be-3ba9-4d3b-9ca8-6e9b44df4c08)<br/>
  Fig. 1.4 dmesg and find camera's boot log<br/>
  <br/>
  Specify the capture device you just get and start gstreamer to get video stream on screen :<br/>
  ` gst-launch-1.0 v4l2src device=/dev/video2 io-mode=2 ! video/x-raw,width=640,height=480 ! waylandsink`
  <br/>
  <br/>
  <br/>

  

  
  
