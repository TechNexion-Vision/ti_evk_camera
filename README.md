# TI TDA4VM Camera Driver

[![TEVS_Banner](https://github.com/user-attachments/assets/02219e99-b031-4a44-84c3-75277ed1a4ec)](https://www.technexion.com/products/embedded-vision/)


[![Producer: TechNexion](https://img.shields.io/badge/Producer-Technexion-blue.svg)](https://www.technexion.com)
[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
<br/><br/>

## Overview
Driver has been developed by TechNexion as an initiative in order to bootup TechNexion MIPI-CSI2 camera modules on Ti starter kit.

## Prerequisites

1. ### **Supported Ti Developer Kit**： <br/>
   * [**SK-AM62**](https://www.ti.com/tool/SK-AM62)<br/>
   * [**SK-AM62A-LP**](https://www.ti.com/tool/SK-AM62A-LP)<br/>
   * [**SK-TDA4VM**](https://www.ti.com/tool/SK-TDA4VM)<br/>
  
2. ### **Adapter board + Cable**： <br/>
   With different board you need prepare match adapter board and cable for the connect pins.<br/>
   Here is the table for you can easily to confirm what you need for your solution.<br/>
   | **Developer Kit**	 | **Adapter board** | **Camera Kit** |
   | --- | --- | --- |
   | SK-AM62	 | TEV-RPI15 Adaptor	 | [TechNexion Camera EVK RPI15](https://www.technexion.com/?s=RPI15&post_type=product) |
   | SK-AM62A-LP	 | TEV-RPI22 Adaptor | [TechNexion Camera EVK RPI22](https://www.technexion.com/?s=RPI22&post_type=product) |
   | SK-TDA4VM	 | TEV-RPI15 Adaptor	 | [TechNexion Camera EVK RPI15](https://www.technexion.com/?s=RPI15&post_type=product) |

   Be careful with the RPI22 adaptor board connector, which is compatible with the reverse orientation pin define.
   <br/>
   Watch this video to make sure your camera is connected correctly :<br/>
   [![Watch the video](https://img.youtube.com/vi/xkC3rHdgO_I/hqdefault.jpg)](https://youtu.be/xkC3rHdgO_I?si=UIqmwzrvaCOmkBf4)
   <br/>
   **Fig. 1.1** The TEV-RPI22 adaptor connection with SK-AM62A-LP Starter Kit.
   <br/><br/>
   The SK-TDA4VM board provides 2 RPi headers for camera connection.<br/>
   <img src="https://github.com/TechNexion-Vision/ti_evk_camera/assets/57210123/7c4a5deb-a138-456f-9716-cf6ed082f93a" width="50%"><br/>
   **Fig. 1.2** The TEV-RPI15 adaptor connection with SK-TDA4VM Starter Kit for Edge AI.
   <br/><br/>
3. ### **Yocto Pre-built image** <br/>
   The release driver have been verified with those ti-processor-sdk-linux version:
   * For **SK-AM62** :<br/>
    [tisdk-default-image-am62xx-evm.wic.xz](https://dr-download.ti.com/software-development/software-development-kit-sdk/MD-PvdSyIiioq/09.01.00.08/tisdk-default-image-am62xx-evm.wic.xz)
      <sub>(Version: 09.01.00.08 Release date: 18 Dec 2023)</sub><br/>
    `xz -d tisdk-default-image-am62xx-evm.wic.xz`<br/>
   * For **SK-AM62A-LP** :<br/>
    [tisdk-edgeai-image-am62axx-evm.wic.xz](https://dr-download.ti.com/software-development/software-development-kit-sdk/MD-D37Ls3JjkT/09.01.00.07/tisdk-edgeai-image-am62axx-evm.wic.xz)
      <sub>(Version: 09.01.00.07 Release date: 18 Dec 2023)</sub><br/>
     `xz -d tisdk-edgeai-image-am62axx-evm.wic.xz`<br/>
   * For **SK-TDA4VM** :<br/>
    [tisdk-edgeai-image-j721e-evm.wic.xz](https://dr-download.ti.com/software-development/software-development-kit-sdk/MD-4K6R4tqhZI/09.01.00.06/tisdk-edgeai-image-j721e-evm.wic.xz)
      <sub>(Version: 09.01.00.06 Release date: 08 Dec 2023)</sub><br/>
     `xz -d tisdk-edgeai-image-j721e-evm.wic.xz`<br/>

> [!NOTE]
> If you don't have any specific driver need, you can using **[TechNexion Pre-built Image](https://tn-docusaurus.vercel.app/docs/embedded-vision/tevs/usage-guides/ti/ti-sk-kit-tevs-camera-usage-guide)**<br/>
> Skip step of driver install and start to using camera quickly !<br/>

4. ### **Micro-SD Card** <br/>
   The capacity must be higher 16GB and then flash the image.<br/>
   `sudo dd of=/dev/<sd card> if=<Yocto prebuild image> bs=1M status=progress`
   <br/><br/>
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

## Instructions for Testing Camera

1. ### Connect debug console cable to SK-TDA4VM.
2. ### Power on the board :
   The display will shown a Ti demo application(Edge AI demo) wallpaper as below.<br/>
   <img src="https://github.com/TechNexion-Vision/ti_evk_camera/assets/57210123/8146a947-d875-4c9f-96de-4df19003d1db" width="400" height="250"><br/>
   **Fig. 1.3** SK-TDA4VM Starter Kit wallpaper upon boot<br/>
> [!NOTE]
> The Ti demo application doesn't include TechNexion's cameras.
> You can stop and disable this by using `systemctl stop edgeai-init.service` and `systemctl disable edgeai-init.service`.<br/>
> You won't see it the next time reboot.

3. ### Checking whether camera have been link via v4l2 control command :
   ```shell
   $ v4l2-ctl --list-device
   ```
   Get the device number of camera capture device

   > #CSI 0 (cam1)<br/>
   > j721e-csi2rx (platform:4504000.ticsi2rx):<br/>
   >   /dev/video{X}<br/>
   ><br/>
   > #CSI 1 (cam2)<br/>
   > j721e-csi2rx (platform:4510000.ticsi2rx):<br/>
   >   /dev/video{X}<br/>

   The following is an example.<br/><br/>
   <img src="https://github.com/user-attachments/assets/16646398-639c-45c3-b9be-4d7b9bbb95eb" width="30%"><br/>
4. ### Get video stream on the display :<br/>
   Specify the capture device you just get (/dev/video2) and start gstreamer to get video stream with 640x480 on the display :
   ```shell
   $ gst-launch-1.0 v4l2src device=/dev/video2 io-mode=2 ! video/x-raw,width=640,height=480 ! waylandsink sync=false
   ```
   
## Troubleshooting

> [!NOTE]
> **Usecase**<br/>
> Take TEVS-AR0234 on SK-TDA4VM CAM2 connector for example:

  ### Find out the Camera<br/>
  <br/>
  Boot up SK-TDA4VM and check initialization of camera driver.<br/>
  It shows below messages, the driver is initialized correctly. If the drivers haven't been probed, please check the connector plugin correct.<br/>
  <br/>
  
   ```shell
   $ dmesg -t | grep "tevs"
   ```

   The message similar to the following :<br/>

   > root@tda4vm-sk:/opt/edgeai-gst-apps# dmesg|grep tevs<br/>
   > tevs 7-0048: tevs_probe() device node: tevs@48<br/>
   > tevs 7-0048: Version:24.8.0.1<br/>
   > tevs 7-0048: Product:TEVS-AR0234, HeaderVer:3, MIPI_Rate:800<br/>
   > tevs 7-0048: probe success<br/>

   Try to using media-ctl framework to print the device topology from camera to CSI and the end to video device node.

   ```shell
   $ media-ctl -d /dev/media1 -p
   ```
   Get the v4l2-subdev which is camera in this print message and then we can use v4l2-ctl by it.

   <img src="https://github.com/user-attachments/assets/4629d202-42c0-42cb-b843-b402fe06251c" width="60%"><br/>

  ### Change Resolution
  1. #### Check camera supported resolutions via v4l2-ctl<br/>
      For kernel 6.1 architecture you cannot easily change resolution in gsteamer pipeline. You must need to using `v4l2-ctl` to do this.
  
      ```shell
      $ v4l2-ctl -d /dev/v4l-subdev2 --list-subdev-framesize pad=0
      ```
      > root@tda4vm-sk:/opt/edgeai-gst-apps# v4l2-ctl -d /dev/v4l-subdev2 --list-subdev-framesize pad=0<br/>
      > ioctl: VIDIOC_SUBDEV_ENUM_FRAME_SIZE (pad=0,stream=0)<br/>
      > Size Range: 640x480 - 640x480<br/>
      > Size Range: 1280x720 - 1280x720<br/>
      > Size Range: 1920x1080 - 1920x1080<br/>
      > Size Range: 1920x1200 - 1920x1200<br/>

  2. #### Set Format
      ```shell
       media-ctl --set-v4l2 "'tevs 7-0048':0 [fmt:UYVY8_2X8/1280X720@1/30 field:none]"
      ```

  <br/>
  <br/>
  <br/>
