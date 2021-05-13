# Cross compilation with Docker on Debian Stretch

## Preliminaries

### ARMv7l (e.g. BeagleBone)
The latest official version of Debian as of 2021 May: __10.3 Buster__.

For Beaglebone and other ARMHF (armv7l) systems: pull the following image:
```bash
docker pull arm32v7/debian:buster
```
Start the VM:
```bash
docker run --rm -it -v /home/kyberszittya/robotlabor/ros2-cross-vm-ws:/home/worker/cross-ws -w / debian:buster
```

## Instructions to setup build system

### Prerequisites
Create a worker user (preferably do not compile ANYTHING with root user). You shall not need password right now.
```bash
adduser --disabled-password --gecos "" worker
```
Very important packages:
```
apt update && apt install -y curl gnupg2 lsb-release
```


Locale settings:
```
locale  # check for UTF-8

apt update && sudo apt install -y locales
locale-gen en_US en_US.UTF-8
update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale
```

Add ROS2 repository:
```bash
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -
sh -c 'echo "deb [arch=$(dpkg --print-architecture)] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros2-latest.list'
```

Install ROS2 preliminaries:
```bash
apt install -y \
  build-essential \
  cmake \
  git \
  libbullet-dev \
  python3-colcon-common-extensions \
  python3-flake8 \
  python3-pip \
  python3-pytest-cov \
  python3-rosdep \
  python3-setuptools \
  python3-vcstool \
  wget
```

Install required Python3 packages:
```bash
python3 -m pip install -U \
  argcomplete \
  flake8-blind-except \
  flake8-builtins \
  flake8-class-newline \
  flake8-comprehensions \
  flake8-deprecated \
  flake8-docstrings \
  flake8-import-order \
  flake8-quotes \
  pytest-repeat \
  pytest-rerunfailures \
  pytest
```
Additional packages:
```bash
apt install --no-install-recommends -y \
  libasio-dev \
  libtinyxml2-dev
```
Numerical packages:
```bash
apt install -y libeigen3-dev liblog4cxx-dev python3-numpy
```
Install lark:
```bash
python3 -m pip install lark
```

### Compiling ROS2
Change to user __worker__:
```bash
su worker
```
Then go to the shared workspace directory:
```bash
cd ~/cross_ws/
mkdir -p ./ros2_foxy/src
cd ./ros2_foxy
wget https://raw.githubusercontent.com/ros2/ros2/foxy/ros2.repos
vcs import src < ros2.repos
```
You can ignore ROS-bridge, visualization tools, rviz2:
```bash
touch src/ros2/rviz/COLCON_IGNORE && touch src/ros2/rviz/AMENT_IGNORE
touch src/ros2/demos/COLCON_IGNORE && touch src/ros2/demos/AMENT_IGNORE
touch src/ros/ros_tutorials/COLCON_IGNORE && touch src/ros/ros_tutorials/AMENT_IGNORE
touch src/ros2/ros1_bridge/COLCON_IGNORE && touch src/ros2/ros1_bridge/AMENT_IGNORE
touch src/ros-visualization/COLCON_IGNORE && touch src/ros-visualization/AMENT_IGNORE
```
You can ignore Cyclone and Connext if you go over FastRTPS:
```bash
touch src/ros2/rmw_connext/COLCON_IGNORE && touch src/ros2/rmw_connext/AMENT_IGNORE
touch src/ros2/rmw_cyclonedds/COLCON_IGNORE && touch src/ros2/rmw_cyclonedds/AMENT_IGNORE
touch src/eclipse-cyclonedds/COLCON_IGNORE && touch src/eclipse-cyclonedds/AMENT_IGNORE
```
Ignore __mimick_vendor__, it won't compile on ARMv7l:
```bash
touch src/ros2/mimick_vendor/COLCON_IGNORE && touch src/ros2/mimick_vendor/AMENT_IGNORE
```

Start compiling the workspace. In this case, ensure __--merge-install__ is used: 
```bash
colcon build --merge-install --cmake-args -DBUILD_TESTING=OFF
```

## Additional packages on BeagleBone
Install the following Python packages on the BeagleBone:
```bash
sudo apt install -y python3-netifaces python3-yaml
```
Install the following C++ packages:
```
sudo apt install -y libtinyxml2-dev
```

## Variable setup
If you encounter problem with __RMW_IMPLEMENTATION_VARIABLE__, you can use the following setup:
```bash
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```