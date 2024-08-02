## General Information
Driver ROS for the RedEdgeMX


<img src="Include/Images/RedEdgeMX.jpeg" alt="RedEdgeMX device" width="600"/> <br/>

> [!IMPORTANT]
> The official contents can be find at the following link:
>   - [ ] [https://support.micasense.com/hc/en-us/articles/360011389334-RedEdge-MX-Integration-Guide](https://support.micasense.com/hc/en-us/articles/360011389334-RedEdge-MX-Integration-Guide)


Response Calib

# Calibration Procedures: zero to calibrated


##  1.0 Pattern Generation 

Pattern generated from:

[calib.io](https://calib.io/pages/camera-calibration-pattern-generator)

**Pattern Kalibr/AprilGrid**

- Board Width [mm] : 420
- Board Height [mm] : 297
- Rows: 5
- Columns: 7
- Tag Size [mm]: 40
- Start id : 0

## 2 launch test_dso container
Pre-operations:

   1. Connect the usb camera to the host computer
   2. Launch the dso container with the bluefox ros node installed

Record a rosbag using the test_dso container:

1. Enable the device
    ``` bash
    udevadm test $(udevadm info -q path -n /dev/bus/usb/004/xxx) 2>&1
    ```
   Replace xxx with the device id of the camera

2. Launch the ros node
    ``` bash
    rosrun bluefox_mono bluefox_mono_node
    ```

## 3 launch kalibr container

``` bash
source /ros_entrypoint.sh && \
kalibr_camera_focus --topic /bluefox_camera/image_raw
```

``` bash
kalibr_calibrate_cameras --bag /root/calibration/calibr_test_1.bag --target /root/calibration/april_6x6_88x88cm.yaml --models pinhole-fov pinhole-fov --topics /bluefox_camera/image_raw /bluefox_camera/image_raw --approx-sync 0.04 --show-extraction
```



config_path=/root/catkin_ws/src/rosbagprocessingTUM/include/calibration/Bluefox_mono/1012bC-2112/fisheye_noname/Tum_Dataset_format/pinhole_fov_model


rosrun dso_ros dso_live image:=/bluefox_camera/image_raw calib=$config_path/camera.txt gamma=$config_path/pcalib.txt vignette=$config_path/vignette.png