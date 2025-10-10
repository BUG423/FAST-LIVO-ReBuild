## 1.ROS安装(Ubuntu 18.04)

### 配置Ubuntu软件仓库

### 1.1.设置sources.list

设置电脑以安装来自packages.ros.org的软件

```text
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

```

![](https://secure2.wostatic.cn/static/i2KcFSUHwzFHt9PyFiBwWD/image.png?auth_key=1760085537-xvVstCktLVcmdsePsczKHQ-0-d2dbee9f5b161e2ba026caef84c6a7a2)

`无返回`

### 1.2.设置密钥

```text
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

![](https://secure2.wostatic.cn/static/6Km4qQ8U1zB2ytp3zM5TUF/image.png?auth_key=1760085537-eQz9c54dMp8yBFNWg1uivE-0-88d251f6553ee9a9a2fcb956fa20a2fc)

### 1.3.安装

首先，确保你的 Debian 包索引是最新的：

```text
sudo apt update
```

在 ROS 中有很多不同的库和工具。我们提供了四种默认选项供你开始。你也可以单独安装 ROS 的软件包。

- **桌面完整版（推荐）：** : 包含 ROS、[rqt](http://wiki.ros.org/rqt)、[rviz](http://wiki.ros.org/rviz)、机器人通用库、2D/3D 模拟器、导航以及 2D/3D 感知包。

```text
sudo apt install ros-melodic-desktop-full
```
- 要查找可用软件包，请运行:

```text
apt search ros-melodic
```

### 1.4.设置环境

将 ROS 环境变量自动添加到新 bash 会话会很方便：

```text
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 1.5.构建依赖

到目前为止，你已经安装了运行核心 ROS 包所需的内容。为了创建和管理自己的 ROS 工作区，有各种各样的工具和需求分别分布。例如：[rosinstall](http://wiki.ros.org/rosinstall) 是一个经常使用的命令行工具，它使你能够轻松地从一个命令下载许多 ROS 包的源树。

要安装这个工具和其他构建ROS包的依赖项，请运行:

```text
sudo apt-get install python-rosinstall python-rosinstall-generator python-wstool build-essential
```

### 1.6.初始化 rosdep

在你使用 ROS 之前，需要初始化 rosdep。rosdep 让你能够轻松地安装被想要编译的源代码，或被某些 ROS 核心组件需要的系统依赖。

```text
sudo apt install python-rosdep
sudo rosdep init
rosdep update
```



# 2.记录

毕设的重点是港大FAST-LIVO算法的移植，原项目是基于ROS开发的。记录下实践过程

## 2.1论文

![](https://secure2.wostatic.cn/static/ahFtXL1WvF7WaQbLujcJbF/image.png?auth_key=1760085537-ey4Uv1CMdBPK43ZD283avU-0-8b5c4cc189888668d9ce532d954a7262)


## 2.2 完善官方文档

  参照该项目的`README.md`,首先是安装常用的三个依赖（科学上网）

  - **Ubuntu and **[**ROS**](http://wiki.ros.org/ROS/Installation)

      建议Ubuntu 16.04-20.04

      为啥没有更新的Ubuntu？因为官方ROS只支持到20.04，之后都是ROS2了

      ROS用`desktop-full`，不然之后有些功能包还得另外安装

      安装功能包指令也放一下：`sudo apt install ros-noetic-PACKAGE`

      我用的是**Ubuntu20.04 & ROS(noetic)**
  - [**PCL**](https://pointclouds.org/downloads/)

      直接`sudo apt install libpcl-dev`

      亲测从源码编译费力不讨好，还容易报错
  - [**Eigen**](https://eigen.tuxfamily.org/index.php?title=Main_Page)

      直接下载`Eigen3.4.0`源码然后编译，**千万别用其他版本**
  - **OpenCV**

      `sudo apt install libopencv-dev python3-opencv`

      亲测从源码编译费力不讨好，还容易报错

      **OpenCV4.0之前的版本用的是****`CV_`****的方式；4.0以后用的是****`cv：：`****，待会如果因为版本不同导致报了这个错误，直接查找替换即可**
  - **Sophus**

      这个比较特殊，要装非模板类的

```text
git clone https://github.com/strasdat/Sophus.git
cd Sophus
git checkout a621ff

```

      到这里停一下，进`Sophus/sophus/so2.cpp`，把

```C++
SO2::SO2()
{
  unit_complex_.real() = 1.;
  unit_complex_.imag() = 0.;
}
```

      替换成

```C++
SO2::SO2()
{
  // unit_complex_.real() = 1.;
  unit_complex_.real(1.);
  // unit_complex_.imag() = 0.;
  unit_complex_.imag(0.);
}
```

      **原因：实例unit_complex_的方法real()和imag()不能用=直接赋值**

      然后继续执行

```C++
mkdir build && cd build && cmake ..
make
sudo make install
```
  - **Vikit**

```C++
cd catkin_ws/src（没这个目录就自己建）
git clone https://github.com/uzh-rpg/rpg_vikit.git
```
  - **livox_ros_driver**
      - Compile Livox SDK

```C++
git clone https://github.com/Livox-SDK/Livox-SDK.git
cd Livox-SDK
cd build && cmake ..
make
sudo make install
```
      - **livox_ros_driver**

```C++
git clone https://github.com/Livox-SDK/livox_ros_driver.git ws_livox/src
cd ws_livox
catkin_make
echo "source ~/ws_livox/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```
  - **Build（终极）**

```C++
cd ~/catkin_ws/src
git clone https://github.com/hku-mars/FAST-LIVO
cd ../
catkin_make
source ~/catkin_ws/devel/setup.bash
```

  版本搞对，基本没啥大问题了，到此就功德圆满了

  测试一下

```C++
roslaunch fast_livo mapping_avia.launch
rosbag play YOUR_DOWNLOADED.bag
```

## 2.3 分析bag内容

  执行`rosbag play YOUR_DOWNLOADED.bag`时，通过rqt和rostopic查看发布的话题及内容

  - 话题

      ![](https://secure2.wostatic.cn/static/rwHbH4fY7CaE8V4Yh5NjX8/image.png?auth_key=1760085695-vhVWHg9C81jykgWtnYEK6j-0-7a26cc23c639c762aac78df50a983a0d)

      ![](https://secure2.wostatic.cn/static/9dpFz2uTJ7JECzDmxfLL9d/image.png?auth_key=1760085695-mj2v9rkNTefRfxUqiMwKhp-0-cb1414956256043634a0946d46a3bc7c)
  - 查看话题数据内容
      - 雷达话题数据

          ![](https://secure2.wostatic.cn/static/q1dAxbe7z87mdfJD8wyJLp/image.png?auth_key=1760085695-XUzrRsvQuqcUGqpuyvT9P-0-fe03b4e15afe814da7479915a54dd069)

          - tag

              ![](https://secure2.wostatic.cn/static/bqsEAQPVtVurU8sVR3rHQa/image.png?auth_key=1760085695-q2ysx1sdHYCGWzmLR44QMc-0-d9b66f0d0aaffea706cc3446060a24e1)

              ![](https://secure2.wostatic.cn/static/937EnVCknDtkQ8LYKCsnXQ/image.png?auth_key=1760085695-99fH5wSiK5pQUMsbeoRNys-0-715ab707862f26de252adf6f6e74fe7f)
          - reflectivity
          - offect_time

              时间偏置（时间戳）
          - x,y,z

              ![](https://secure2.wostatic.cn/static/8XkuEEVNqdjdStohn4kiHX/image.png?auth_key=1760085695-swLkPRj1mvHNwaXPYGgtx4-0-9afde2a940b5d2ea945aa144a21b5414)
          - line（只有0-5）

              最后一行**line**是激光雷达扫描的线数（livox avia为6线），从原始扫描点云可以看出，后期可以按线来对原始点处理、筛选
      - imu话题数据

          ![](https://secure2.wostatic.cn/static/wb7qvvDKhjYwHGwMkGzYh/image.png?auth_key=1760085695-eL3Bz94AfKovDKGZtGYkxK-0-b6bdc0b660d0dec81c46dd73450e55d2)
  - bag包整体数据情况

      ![](https://secure2.wostatic.cn/static/5KDG7TvENkVrzaZ4GqFNN5/image.png?auth_key=1760085695-gcFMyk8vRrwW3r4v6StLUS-0-66f44e327e37a03e993cc067941b155d)

      可以看到，IMU的频率高于摄像头和雷达数据；摄像头和雷达数据严格同步，10HZ

  https://blog.csdn.net/weixin_43455581/article/details/96168042

## 2.4 硬件处理

  ### 镭神C16雷达

  ![](https://secure2.wostatic.cn/static/wzHhqfogRN5iQBtfVsGxtX/image.png?auth_key=1760085700-oTgxHQQiVkTfwN2iyDwbpu-0-6cb0459a6c96f8eecaafda8e2db62568)

  在驱动代码中找到了x,y,z和时间戳的计算，用指针将其读出来，然后发布自己的话题（话题格式参考livox/lidar）

  - 定义自己的msg格式

      [(49条消息) ros中自定义msg消息并用其他功能包调用_ros 自定义msg_我是你们队长阿威啊的博客-CSDN博客](https://blog.csdn.net/zhou2677273778/article/details/126477921)

      https://blog.csdn.net/prolop87/article/details/124594364

      ![](https://secure2.wostatic.cn/static/woML8jrVB9bJJ8giNEmqfX/image.png?auth_key=1760085700-9Ma7Q7v613t8kM4U4Px2K7-0-d0f5391b67e449f753a683fbe3fd27ac)
  - 解析雷达数据包，并提取有用信息发布
      - 指针

```C++
    double *x;
    double *y;
    double *z;
    uint32_t *time_stamp;
    x = (double *)malloc(sizeof(double) * 3);
    y = (double *)malloc(sizeof(double) * 3);
    z = (double *)malloc(sizeof(double) * 3);
    time_stamp = (uint32_t *)malloc(sizeof(uint32_t) * 3);
```
      - 提取有用数据

          对虚函数的修改卡了很久，请教了mz和奇神；自己对C++还是不熟，需要再多写代码

```C++
        // packet timestamp
        if (use_gps_ts)
        {
            lslidar_msgs::LslidarPacket pkt = *packet;
            memset(&cur_time, 0, sizeof(cur_time));
            if (packet_size == 1206)
            {
                cur_time.tm_year = this->packetTimeStamp[9] + 2000 - 1900;
                cur_time.tm_mon = this->packetTimeStamp[8] - 1;
                cur_time.tm_mday = this->packetTimeStamp[7];
                cur_time.tm_hour = this->packetTimeStamp[6];
                cur_time.tm_min = this->packetTimeStamp[5];
                cur_time.tm_sec = this->packetTimeStamp[4];
                packet_time_s = static_cast<uint64_t>(timegm(&cur_time)); // s
                packet_time_ns = (pkt.data[1200] +
                                  pkt.data[1201] * pow(2, 8) +
                                  pkt.data[1202] * pow(2, 16) +
                                  pkt.data[1203] * pow(2, 24)) *
                                 1e3; // ns
                timeStamp = ros::Time(packet_time_s, packet_time_ns);
                // 使用gps授时
                if ((timeStamp - timeStamp_bak).toSec() < 0.0 && (timeStamp - timeStamp_bak).toSec() > -1.0 &&
                    packet_time_ns < 100000000 && is_update_gps_time)
                {
                    timeStamp = ros::Time(packet_time_s + 1, packet_time_ns);
                }
                else if ((timeStamp - timeStamp_bak).toSec() > 1.0 && (timeStamp - timeStamp_bak).toSec() < 1.2 && packet_time_ns > 900000000 && is_update_gps_time)
                {
                    timeStamp = ros::Time(packet_time_s - 1, packet_time_ns);
                }
                timeStamp_bak = timeStamp;
                packet->stamp = timeStamp;
                packet_timestamp = packet->stamp.toSec();
            }
            else if (packet_size == 1212)
            {
                cur_time.tm_year = pkt.data[1200] + 2000 - 1900;
                cur_time.tm_mon = pkt.data[1201] - 1;
                cur_time.tm_mday = pkt.data[1202];
                cur_time.tm_hour = pkt.data[1203];
                cur_time.tm_min = pkt.data[1204];
                cur_time.tm_sec = pkt.data[1205];
                packet_time_s = static_cast<uint64_t>(timegm(&cur_time)); // s
                packet_time_ns = (pkt.data[1206] +
                                  pkt.data[1207] * pow(2, 8) +
                                  pkt.data[1208] * pow(2, 16) +
                                  pkt.data[1209] * pow(2, 24)) *
                                 1e3; // ns
                packet_timestamp = packet_time_s + packet_time_ns * 1e-9;
                timeStamp = ros::Time(packet_time_s, packet_time_ns);
                packet->stamp = timeStamp;
            }
        }
        else
        {
            packet->stamp = ros::Time::now();
            packet_timestamp = packet->stamp.toSec();
        }

        packet_interval_time = (packet_timestamp - last_packet_timestamp) / 384.0;
        last_packet_timestamp = packet_timestamp;
        if (packet_size == 1206)
        {
            if (packet->data[1204] == 0x39)
            {
                return_mode = 2;
            }
        }
        else if (packet_size == 1212)
        {
            if (packet->data[1210] == 0x39)
            {
                return_mode = 2;
            }
        }

        ROS_INFO_ONCE("return mode: %d", return_mode);
        // decode the packet
        decodePacket(raw_packet);
        ROS_INFO_ONCE("decode succ distance and intensity: ");
        // find the start of a new revolution
        // if there is one, new_sweep_start will be the index of the start firing,
        // otherwise, new_sweep_start will be FIRINGS_PER_PACKET.
        size_t new_sweep_start = 0;
        do
        {
            if (last_azimuth - firings.azimuth[new_sweep_start] > 35000)
            {
                break;
            }
            else
            {
                last_azimuth = firings.azimuth[new_sweep_start];
                ++new_sweep_start;
            }
        } while (new_sweep_start < SCANS_PER_PACKET);

        // The first sweep may not be complete. So, the firings with
        // the first sweep will be discarded. We will wait for the
        // second sweep in order to find the 0 azimuth angle.
        size_t start_fir_idx = 0;
        size_t end_fir_idx = new_sweep_start;
        if (is_first_sweep && new_sweep_start == SCANS_PER_PACKET)
        {
            return true;
        }
        else
        {
            if (is_first_sweep)
            {
                is_first_sweep = false;
                start_fir_idx = new_sweep_start;
                end_fir_idx = SCANS_PER_PACKET;
                // sweep_start_time = packet->stamp.toSec() - (end_fir_idx - start_fir_idx) * 3.125 * 1e-6;
            }
        }
        for (size_t fir_idx = start_fir_idx; fir_idx < end_fir_idx; ++fir_idx)
        {
            // check if the point is valid
            if (!isPointInRange(firings.distance[fir_idx]))
                continue;
            // convert the point to xyz coordinate
            size_t table_idx = firings.azimuth[fir_idx];
            double cos_azimuth = cos_azimuth_table[table_idx];
            double sin_azimuth = sin_azimuth_table[table_idx];
            double x_coord, y_coord, z_coord;
            if (coordinate_opt)
            {
                int tmp_idx = 1468 - firings.azimuth[fir_idx] < 0 ? 1468 - firings.azimuth[fir_idx] + 36000 : 1468 - firings.azimuth[fir_idx];
                x_coord = firings.distance[fir_idx] * c16_cos_scan_altitude[fir_idx % 16] * cos_azimuth +
                          R1_ * cos_azimuth_table[tmp_idx];
                y_coord = -firings.distance[fir_idx] * c16_cos_scan_altitude[fir_idx % 16] * sin_azimuth +
                          R1_ * sin_azimuth_table[tmp_idx];
                z_coord = firings.distance[fir_idx] * c16_sin_scan_altitude[fir_idx % 16];
            }
            else
            {
                // Y-axis correspondence 0 degree
                int tmp_idx = firings.azimuth[fir_idx] - 1468 < 0 ? firings.azimuth[fir_idx] - 1468 + 36000 : firings.azimuth[fir_idx] - 1468;
                x_coord = firings.distance[fir_idx] * c16_cos_scan_altitude[fir_idx % 16] * sin_azimuth +
                          R1_ * sin_azimuth_table[tmp_idx];
                y_coord = firings.distance[fir_idx] * c16_cos_scan_altitude[fir_idx % 16] * cos_azimuth +
                          R1_ * cos_azimuth_table[tmp_idx];
                z_coord = firings.distance[fir_idx] * c16_sin_scan_altitude[fir_idx % 16];
            }
            // computer the time of the point
            double time = packet_timestamp -
                          (SCANS_PER_PACKET - fir_idx - 1) * packet_interval_time - sweep_end_time;
            int aft = int((packet_timestamp - int(packet_timestamp))*1000)+(int(packet_timestamp)%100000)*1000;
            ROS_INFO("packet_timestamp = %f,aft = %f,x = %f, y = %f, z = %f", packet_timestamp,aft,x_coord, y_coord, z_coord); // print time x,y,z                                                     // 成员变量赋值
            *x = x_coord; //
            *y = y_coord; //
            *z = z_coord; //
            *time_stamp = aft; //
```
      - 发布话题

```C++
            Livox::CustomPoint livoxPoint;
            livoxPoint.x = *x;
            livoxPoint.y = *y;
            livoxPoint.z = *z;
            livoxPoint.tag = 0;
            livoxPoint.line = 0;
            livoxPoint.offset_time = *time_stamp;
            pubLivox.publish(livoxPoint);
```
      - 问题

          tag、line和反射率镭神没有提供，置0

  ### IMU（MTI-300）

  https://blog.csdn.net/qq_34911636/article/details/108937265

  - 查看用户组

      `grep 'dialout' /etc/group`

      若结果为`dialout:x:20:`

      则说明root用户或者普通用户没有加入到dialout组中，接下去我们需要把root用户或者自己的用户加入到dialout组

      执行

      `sudo usermod -a -G dialout root`

      `sudo usermod -a -G dialout username `

      然后再次查看：`grep 'dialout' /etc/group`

      输出`dialout:x:20:root,username`即可
  - USB状态权限

      `ls -l /dev/ttyUSB*`

      `sudo chmod 777 /dev/ttyUSB*`
  - 安装驱动

      `sudo apt-get install ros-kinetic-xsens-driver`

      启动`roslaunch xsens_driver xsens_driver.launch`

      查看数据`rostopic echo /ium/data`

      数据格式和livox一致，完美！也可能是IMU的默认数据规范格式

      ![](https://secure2.wostatic.cn/static/pXw99qNwLWM2d9opDXCg7Y/image.png?auth_key=1760085700-vVnVkbqiGixjuv8KyF8Ysk-0-fb72029285370693353b8303f5720679)

  ### ZED2

  zed2相机需要CUDA

  ![](https://secure2.wostatic.cn/static/yYPkULB2LhzEMafSCCq6o/image.png?auth_key=1760085700-noUjFYHib7L2crT8DRLQRu-0-e521cb828a7f6c93486ed30a7d0b3450)

## 2.5其他

- 标定工具

    [https://github.com/ethz-asl/kalibr](https://github.com/ethz-asl/kalibr)

[https://github.com/ethz-asl/lidar_align](https://github.com/ethz-asl/lidar_align)

    
[https://github.com/PJLab-ADG/SensorsCalibration](https://github.com/PJLab-ADG/SensorsCalibration)
- 时间同步

    https://zhuanlan.zhihu.com/p/191372189

    https://blog.csdn.net/qq_24972557/article/details/115049024

    ![](https://secure2.wostatic.cn/static/k1VDVuTcNSxKhaL1xNGM5G/image.png?auth_key=1760085538-oTZ6FN5P847fGTvCozgDuG-0-c65bdba4d5f6974b9d77ea48edc983fa)

    

