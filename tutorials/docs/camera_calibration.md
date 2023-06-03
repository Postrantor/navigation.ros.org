---
tip: translate by openai@2023-06-03 00:16:29
title: Camera Calibration
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)

# Overview

This tutorial shows how to obtain calibration parameters for monocular camera.

> 这篇教程展示了如何获得单目摄像头的校准参数。

# Requirements

1- Install Camera Calibration Parser, Camera Info Manager and Launch Testing Ament Cmake using operating system's package manager:

> 安装相机校准解析器、相机信息管理器和使用操作系统的包管理器启动测试 Ament Cmake：`等待`

> `sudo apt install ros-<ros2-distro>-camera-calibration-parsers` > `sudo apt install ros-<ros2-distro>-camera-info-manager` > `sudo apt install ros-<ros2-distro>-launch-testing-ament-cmake`

2- Image Pipeline need to be built from source in your workspace with:

> 在你的工作空间中，需要从源头构建 2-图像流水线：`Wait`

> `git clone – b <ros2-distro> git@github.com:ros-perception/image_pipeline.git`

**Also, make sure you have the following:**

> **也请确保您拥有以下内容：**

- A large checkerboard with known dimensions. This tutorial uses a 7x9 checkerboard with 20mm squares. **Calibration uses the interior vertex points of the checkerboard, so an \"8x10\" board uses the interior vertex parameter \"7x9\" as in the example below.** The checkerboard with set dimensions can be downloaded from [here](https://calib.io/pages/camera-calibration-pattern-generator).

> 一个尺寸已知的大棋盘。本教程使用 7x9 棋盘，每个方格的尺寸为 20mm。**校准时使用棋盘内部顶点，因此一个“8x10”棋盘使用的内部顶点参数是“7x9”，如下例所示。** 尺寸已设定的棋盘可从[这里](https://calib.io/pages/camera-calibration-pattern-generator)下载。

- A well-lit area clear of obstructions and other check board patterns
- A monocular camera publishing images over ROS

# Tutorial Steps

1- Start a terminal in your GUI

> 开始一个终端在你的 GUI 中 `Wait`

2- Launch the ROS driver for your specific camera.

> 启动针对您特定摄像头的 ROS 驱动程序。

3- Make sure camera is publishing images over ROS. This can be tested by running:

> 确保相机通过 ROS 发布图像。可以通过运行以下命令来测试：`Wait`

> `ros2 topic list`

4- This will show you all the topics published make sure that there is an image_raw topic /camera/image_raw. To confirm that its a real topic and actually publishing check topic hz:

> 这将向您展示所有发布的主题，请确保存在一个名为 image_raw 的主题/camera/image_raw。要确认它是一个真正的主题并且实际发布，请检查 hz 主题：`Wait`

> `ros2 topic hz /camera/image_raw` > ![image](images/Camera_Calibration/ROS2_topic_hz.png){.align-center width="400px" height="300px"}

5- Start the camera calibration node

> 开始相机校准节点

> `ros2 run camera_calibration cameracalibrator --size 7x9 --square 0.02 --ros-args -r image:=/my_camera/image_raw -p camera:=/my_camera`

    Camera Name:

    -c, --camera_name
            name of the camera to appear in the calibration file


    Chessboard Options:

    You must specify one or more chessboards as pairs of --size and--square options.

      -p PATTERN, --pattern=PATTERN
                        calibration pattern to detect - 'chessboard','circles', 'acircles','charuco'
      -s SIZE, --size=SIZE
                        chessboard size as NxM, counting interior corners (e.g. a standard chessboard is 7x7)
      -q SQUARE, --square=SQUARE
                        chessboard square size in meters

    ROS Communication Options:

     --approximate=APPROXIMATE
                        allow specified slop (in seconds) when pairing images from unsynchronized stereo cameras
     --no-service-check
                        disable check for set_camera_info services at startup

    Calibration Optimizer Options:

     --fix-principal-point
                        fix the principal point at the image center
     --fix-aspect-ratio
                        enforce focal lengths (fx, fy) are equal
     --zero-tangent-dist
                        set tangential distortion coefficients (p1, p2) to
                        zero
     -k NUM_COEFFS, --k-coefficients=NUM_COEFFS
                        number of radial distortion coefficients to use (up to
                        6, default 2)
     --disable_calib_cb_fast_check
                        uses the CALIB_CB_FAST_CHECK flag for findChessboardCorners

    This will open a calibration window which highlight the checkerboard.

> ![image](images/Camera_Calibration/window1.png){.align-center width="400px" height="300px"}

6- In order to get a good calibration you will need to move the checkerboard around in the camera frame such that:

> 为了获得良好的校准，您需要在相机框架中移动棋盘，以便：

> - checkerboard on the camera\'s left, right, top and bottom of field of view
>
>   : ◦ X bar - left/right in field of view
>
>       ◦ Y bar - top/bottom in field of view
>
>       ◦ Size bar - toward/away and tilt from the camera
>
> - checkerboard filling the whole field of view
>
> - checkerboard tilted to the left, right, top and bottom (Skew)

> ![image](images/Camera_Calibration/calibration.jpg){.align-center width="400px" height="300px"}

7- As the checkerboard is moved around the 4 bars on the calibration sidebar increases in length. When all then the 4 bars are green and enough data is available for calibration the CALIBRATE button will light up. Click it to see the results. It takes around the minute for calibration to take place.

> ![image](images/Camera_Calibration/greenbars.png){.align-center width="400px" height="300px"}

8- After the calibration is completed the save and commit buttons light up. And you can also see the result in terminal.

> ![image](images/Camera_Calibration/calibration_complete.png){.align-center width="400px" height="300px"}
> ![image](images/Camera_Calibration/calibration_parameters.png){.align-center width="400px" height="300px"}

9-Press the save button to see the result. Data is saved to \"/tmp/calibrationdata.tar.gz\"

10-To use the the calibration file unzip the calibration.tar.gz

: `tar -xvf calibration.tar.gz`

11-In the folder images used for calibration are available and also "**ost.yaml**" and "**ost.txt**". You can use the yaml file which contains the calibration parameters as directed by the camera driver.
