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
# 取得專案
cd ~
git clone https://github.com/<你的帳號>/graduation-project.git
cd graduation-project
專案已加入 .gitignore，會忽略各工作區的 build/ install/ log/。

4. 專案結構與建置
4.1 omx_ws（OpenMANIPULATOR-X）
bash
複製程式碼
# 進到工作區根目錄
cd ~/graduation-project/omx_ws

# 解析依賴（第一次與變更後建議執行）
rosdep install --from-paths src -y --ignore-src

# 建置
colcon build --symlink-install

# 載入環境（每開一個新終端機都要）
source install/setup.bash
常見套件（本 repo 已內含）：
open_manipulator, open_manipulator_msgs, robotis_manipulator

4.2 my_ros2_ws（自建工作區）
bash
複製程式碼
cd ~/graduation-project/my_ros2_ws

# 若 src 有新套件，先補齊依賴
rosdep install --from-paths src -y --ignore-src

# 建置
colcon build --symlink-install

# 載入環境
source install/setup.bash
5. TurtleBot4 Lite 基本使用
下列以筆電端操作為例，請先與機器人連到同一網段。
若需遠端連線，請依你的網路設定補上 ROS_DOMAIN_ID 或 Fast DDS discovery 設定。

5.1 安裝 TurtleBot4 相關套件（若尚未安裝）
bash
複製程式碼
sudo apt install -y ros-humble-turtlebot4-* ros-humble-navigation2 ros-humble-nav2-bringup
5.2 機器人最小啟動（Bringup）
在機器人端（或透過 SSH 到機器人）：

bash
複製程式碼
export TURTLEBOT4_MODEL=lite
ros2 launch turtlebot4_bringup minimal.launch.py
5.3 導航（Nav2）
有現成地圖：

bash
複製程式碼
# 筆電端，載入 my_ros2_ws 的環境
source ~/graduation-project/my_ros2_ws/install/setup.bash

# 啟動導航（map:= 你的地圖檔 .yaml）
ros2 launch turtlebot4_navigation nav2.launch.py map:=/path/to/map.yaml use_sim_time:=false
即時建圖（SLAM）可改用對應的 slam/bringup 方案（依你專案提供的 launch 檔為準）。

6. OpenMANIPULATOR-X 基本使用
6.1 U2D2 規則（選用）
若使用 U2D2，建議新增 udev 規則讓裝置名稱固定，例如：

text
複製程式碼
SYMLINK+="U2D2"
完成後重新插拔裝置，並確認連線埠名稱。

6.2 啟動控制與遙控
bash
複製程式碼
# 載入手臂工作區環境
source ~/graduation-project/omx_ws/install/setup.bash

# 控制器
ros2 launch open_manipulator_x_controller open_manipulator_x_controller.launch.py

# 遙控（鍵盤/GUI 等）
ros2 launch open_manipulator_x_teleop open_manipulator_x_teleop.launch.py
若你的專案提供了自訂的節點或服務（例如夾爪服務 goal_tool_control、關節命令 goal_joint_space_path 等），請在此補充對應的啟動與呼叫方式。

7. 常見指令
bash
複製程式碼
# 重新建置單一工作區
cd ~/graduation-project/omx_ws && colcon build --symlink-install
cd ~/graduation-project/my_ros2_ws && colcon build --symlink-install

# 載入環境（不同終端機都要）
source ~/graduation-project/omx_ws/install/setup.bash
source ~/graduation-project/my_ros2_ws/install/setup.bash

# 檢視可用主題/服務/動作
ros2 topic list
ros2 service list
ros2 action list
8. 專案開發與貢獻
bash
複製程式碼
# 拉取最新
git pull origin main --rebase

# 新增或修改檔案
git add .
git commit -m "說明此次修改"
git push
注意：請不要把 build/ install/ log/ 推上 GitHub。
若誤加，請移除追蹤後再提交：

bash
複製程式碼
git rm -r --cached **/build **/install **/log
git commit -m "clean artifacts"
git push
9. 疑難排解
GitHub 無法展開資料夾且檔案型態顯示 160000
→ 表示被當成 git submodule。請移除子資料夾內的 .git，並在專案根目錄執行：

bash
複製程式碼
git rm --cached <path>
git add .
git commit -m "fix submodule issue"
git push
colcon build 依賴缺失
→ 請先執行：

bash
複製程式碼
rosdep install --from-paths src -y --ignore-src
TurtleBot4 無法連線
→ 確認與機器人同網段、時間同步、必要埠未被防火牆阻擋，並檢查
TURTLEBOT4_MODEL 與對應套件是否安裝完整
