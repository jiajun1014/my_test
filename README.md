# Graduation Project (ROS 2 Humble, TurtleBot4 Lite)

本專案在 **Ubuntu 22.04 + ROS 2 Humble** 環境開發，硬體以 **TurtleBot4 Lite** 為主，並整合 **OpenMANIPULATOR-X**。  
Repository 內含兩個 ROS 2 工作區：

```text
omx_ws/         # 手臂相關套件 (open_manipulator, open_manipulator_msgs, robotis_manipulator)
my_ros2_ws/     # 自建工作區 (自訂套件、turtlebot4_navigation 等)
1. 環境需求
Ubuntu 22.04
```
# Ubuntu 22.04安裝教學：
進入這個網站下載映像檔 - > https://releases.ubuntu.com/jammy/

載完之後進入BIOS進行安裝動作，磁碟分割請依照自行所需來進行分割

進入ubuntu以後再來安裝ROS 2 Humble

# ROS 2 Humble 安裝教學：
```text
$ locale  # check for UTF-8

$ sudo apt update && sudo apt install locales
$ sudo locale-gen en_US en_US.UTF-8
$ sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
$ export LANG=en_US.UTF-8

$ locale  # verify settings

$ sudo apt install software-properties-common
$ sudo add-apt-repository universe
$ sudo apt update && sudo apt install curl -y
$ export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
$ curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo $VERSION_CODENAME)_all.deb" # If using Ubuntu derivates use $UBUNTU_CODENAME
$ sudo dpkg -i /tmp/ros2-apt-source.deb
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install ros-humble-desktop
$ sudo apt install ros-humble-ros-base
$ sudo apt install ros-dev-tools
```
PC都設置好接著就是turtlebot4的部分

樹梅派的映象檔可以從這個網站進行安裝自行所需的版本 -> http://download.ros.org/downloads/turtlebot4/ (需與電腦的版本一致)

接者再從這邊進行基本設置 -> https://turtlebot.github.io/turtlebot4-user-manual/setup/basic.html

# 擴充 Open Manipulator X 手臂
安裝套件 ( 需要 PC & TurtleBot4 皆安裝 )
```text
$ sudo apt install ros-humble-joint-state-publisher ros-humble-ament-cmake ros-humble-dynamixel-sdk ros-humble-dynamixel-workbench
```
到這邊以後就可以將github上的 omx_ws/ & my_ros2_ws/ 裝上去
如果想自行動手嘗試可以繼續裝下去
```text
$ mkdir -p omx_ws/src
$ cd omx_ws/src
$ git clone -b ros2 https://github.com/ROBOTIS-GIT/open_manipulator_msgs.git
$ git clone -b ros2 https://github.com/ROBOTIS-GIT/robotis_manipulator.git
$ git clone -b humble-devel https://github.com/ROBOTIS-GIT/open_manipulator.git
```
開啟 robotis_manipulator/CMakeLists.txt 文件並刪除 20、37、77 行使用 cmake_modules 套件語法，改完之後重新編譯。
```text
$ cd ~/omx_ws && colcon build --symlink-install
$ echo "source ~/omx_ws/install/setup.bash" >> ~/.bashrc
```
TurtleBot4 端執行腳本修改 U2D2 權限及 lantency_timer。
```text
$ ros2 run open_manipulator_x_controller create_udev_rules
```
執行 ( 限於 TurtleBot4 端操作 )
```text
$ ros2 launch open_manipulator_x_controller open_manipulator_x_controller.launch.py
```
遙控 ( 兩端皆可操作 )
```text
$ ros2 run open_manipulator_x_teleop teleop_keyboard
```

# 建置
每個檔案編輯過記得都要重新建置
colcon build --symlink-install

