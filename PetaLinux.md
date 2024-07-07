---
marp: true
theme: uncover
_class: invert
paginate: true
---

<style>
  :root {
    --color-highlight: #EE0000;
    --color-highlight-hover: #aaf;
    --color-highlight-heading: #EE0000;
    --color-header: #bbb;
    --color-header-shadow: transparent;
  }

  h2 {
    position: absolute;
    top: 30px;
    font-size: 1.5rem;
  }

  p {
  	text-align: middle;
  }

  /* pre {
    width:100%;
    font-size: 26px;
  } */
  pre {
    width:100%;
    /* font-size: 26px; */
    /* white-space: break-spaces; */
    white-space: pre-wrap; 
    line-break: anywhere;
  }
</style>

# **Build Platform**

Build PetaLinux OS Image from Scratch

---

## Outline

- step 0: Create a Container Env.
- step 1: Prepare PetaLinux Env.
- step 2: Install PetaLinux Tools
- step 3: Manage a PetaLinux Project
- step 4: Verification

---

## step 0: Create a Container Env.

```sh
# On Host machine. Open X-window permission
$xhost +
# Create a container
$docker run -itd --gpus all --privileged
 -v /tmp/.X11-unix:/tmp/.X11-unix
 -v $PWD:/krs_ws
 -e DISPLAY=$DISPLAY
 --name my_krs_devenv_plnx
 my_kr260_dev /bin/bash
# attach the container
$docker exec -it my_krs_devenv_plnx /bin/bash
```

---

## step 1: Prepare Petalinux Env.

##### 1.1: Complete Vitis Setup

```sh
$bash /tools/Xilinx/Vitis/2022.1/scripts/installLibs.sh
```

---

## step 1: Prepare Petalinux Env. (cont.)

##### 1-2. 安裝 PetaLinux tools 所需的依賴包

```sh
# Required to install zlib1g:i386
# E: Unable to locate package zlib1g:i386

$sudo dpkg --add-architecture i386
$sudo apt-get update
$sudo apt-get install zlib1g:i386

# Installation of various packages (example)
$sudo apt install gawk build-essential net-tools xterm autoconf libtool
 libtinfo5 texinfo  gcc-multilib libncurses5-dev libncursesw5-dev
 zlib1g-dev zlib1g:i386
```

---

## step 1: Prepare Petalinux Env. (cont.)

##### 1-3. 將預設的 Shell 由 dash 改為 bash

- Petalinux tools 預設是使用 bash。

```sh
# Check if /bin/sh is dash or bash
$ ls -l /bin/sh
lrwxrwxrwx 1 root root 4 Sep 11  2020 /bin/sh -> dash

# dash, execute the following command and select "no" in the dialog that appears
$ sudo dpkg-reconfigure dash

# Make sure /bin/sh is bash
$ ls -l /bin/sh
lrwxrwxrwx 1 root root 4 Oct 13  2022 /bin/sh -> bash
```

---

## step 2: Install Petalinux Tools

##### 2-0. 準備 PetaLinux Tools 安裝包。

Download the PetaLinux 2022.1 installer from
[the Xilinx website](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/2022-1.html).

```text
注意：安裝包跟 peta-linux 指令皆不可為 root 用戶
議題：如何在 docker 中使用 non-root 權限
```

---

## step 2: Install Petalinux Tools (cont.)

##### 2-1. 安裝 PetaLinux Tools 安裝包。

###### 2-1-1. 加入 user 身份的使用者 與權限密碼

```sh
# 創建 non-root 用戶
$groupadd -r john && useradd -r -g john john
$chown -R john:john /krs_ws
# 加入權限密碼
$passwd john

# 將用戶加入sudoers
$visudo /etc/sudoers
```

---

###### 2-1-2. 安裝依賴包

```sh
$sudo apt install less rsync bc
```

---

###### 2-1-3. 開始安裝

```sh
## 切換到 non-root 用戶
$su john

$ cd /krs_ws
# Grant execute permission if you don't have them.
$ chmod +x ./petalinux-v2022.1-04191534-installer.run

# install (--dir xxx specifies the directory to install in)
$ ./petalinux-v2022.1-04191534-installer.run --dir ./Petalinux
## 打印訊息：<space> page-down, <q> 退出 作選擇
INFO: Checking installation environment requirements...
INFO: Checking free disk space
INFO: Checking installed tools
INFO: Checking installed development libraries
INFO: Checking network and other services
WARNING: No tftp server found - please refer to
         "UG1144  PetaLinux Tools Documentation Reference Guide"
         for its impact and solution
INFO: Checking installer checksum...
INFO: Extracting PetaLinux installer...

LICENSE AGREEMENTS

PetaLinux SDK contains software from a number of sources.  Please review
the following licenses and indicate your acceptance of each to continue.

You do not have to accept the licenses, however if you do not then you may
not use PetaLinux SDK.

Use PgUp/PgDn to navigate the license viewer, and press 'q' to close

Press Enter to display the license agreements
Do you accept Xilinx End User License Agreement? [y/N] > y
Do you accept Third Party End User License Agreement? [y/N] > y
INFO: Installing PetaLinux...
INFO: Checking PetaLinux installer integrity...
INFO: Installing PetaLinux SDK to "/home/username/Petalinux/."
INFO: Installing buildtools in /home/username/Petalinux/./components/yocto/buildtools
INFO: Installing buildtools-extended in
      /home/username/Petalinux/./components/yocto/buildtools_extended
INFO: PetaLinux SDK has been installed to /home/username/Petalinux/.
```

---

## step 3: Manage a Petalinux Project

##### 創建 Petalinux 專案

```sh
# 溯源 Petalinux 設定腳本
$source ./Petalinux/settings.sh

# 創建專案
# use default name
$petalinux-create
 -t project
 -s xilinx-kr260-starterkit-v2022.1-05140151.bsp
 --name plnx_os
```

---

## step 3: Manage a Petalinux Project

##### 先備工作: fix locale issue

```sh
# as a non-root user
$sudo apt-get install locales
$sudo dpkg-reconfigure locales
$sudo locale-gen
# --- 輸出訊息 ---
Generating locales (this might take a while)...
en_US.UTF-8... done
Generation complete.
# --------------

# or
$sudo apt-get install locales
sudo locale-gen en_US.UTF-8
```

---

##### 3-1. 創建 Petalinux 專案

- 配置專案

```sh
$cd <PLNX_PROJ_DIR> (eg. plnx_os, xilinx-k26-som-v2022.1)
## 使用默認設定，連線進行設置。
$petalinux-config --silentconfig
```

---

##### 離線配置

```sh
$mkdir -p /krs_ws/assets

# 解壓
$tar -xzvf /krs/assets/sstate_aarch64_2022.1_09151236.tar.gz
$tar -xzvf /krs/assets/downloads_2022.1_09151236.tar.gz
$petalinux-config

# setup petalinux-config
Yocto Settings > Add pre-mirror url > file:///krs_ws/assets/downloads
Yocto Settings > Local sstate feeds settings > file:///krs_ws/assets/aarch64

[*] Enable BB NO NETWORK
```

---

##### 在 ROS2 Humble 加入後設層(meta-layers)，並於 Yocto 中 Petalinux 配置後設層。

```sh
$cd <PLNX_PROJ_DIR> (eg. plnx_os, xilinx-k26-som-v2022.1)
$git clone https://github.com/vmayoral/meta-ros -b honister-humble project-spec/meta-ros
# or
$tar -xzvf ../assets/meta_ros_honister-humble.tar.gz ./project-spec/meta-ros
```

---

##### 修改 bblayers.conf 檔

```sh
$vim /krs_ws/<PLNX_PROJ_DIR>/build/conf/bblayers.conf
# eg.
$vim /krs_ws/plnx_os/build/conf/bblayers.conf
```

---

頭加上

```text
LCONF_VERSION = "7"
# --- 加入部份 ---
# define the ROS 2 Yocto target release
ROS_OE_RELEASE_SERIES = "honister"

# define ROS 2 distro
ROS_DISTRO = "humble"

...
```

---

尾加上

```text
  ${SDKBASEMETAPATH}/layers/meta-security \
  ${SDKBASEMETAPATH}/layers/meta-security/meta-tpm \
  /krs_ws/xilinx-k26-som-2022.1/project-spec/meta-user \
  /krs_ws/xilinx-k26-som-2022.1/components/yocto/workspace \
  ###### --- 加入部份 --- ######
  ${SDKBASEMETAPATH}/../../project-spec/meta-ros/meta-ros2-humble \
  ${SDKBASEMETAPATH}/../../project-spec/meta-ros/meta-ros2 \
  ${SDKBASEMETAPATH}/../../project-spec/meta-ros/meta-ros-common \
```

---

### Extend ROS2 in Yocto's Minimum Image

<!-- 在 Yocto's Minimum Image 中擴展 ROS2 需要的內容 -->

```sh
$cd /krs_ws/<PLNX_PROJ_NAME>(eg. plnx_os, xilinx-k26-som-2022.1)
# 創建路徑
$mkdir -p project-spec/meta-user/recipes-images/images
# 擴展預設的Petalinux映像檔配方(Petalinux image recipe, petalinux-image-minimal.bb)
$cat << 'EOF' > project-spec/meta-user/recipes-images/images/petalinux-image-minimal.bbappend
require ${COREBASE}/../meta-petalinux/recipes-core/images/petalinux-image-minimal.bb

SUMMARY = "A image including a bare-minimum installation of ROS 2 and including some basic pub/sub examples. It includes two DDS middleware implementations, FastDDS and Cyclone DDS"
DESCRIPTION = "${SUMMARY}"

inherit ros_distro_${ROS_DISTRO}
inherit ${ROS_DISTRO_TYPE}_image

ROS_SYSROOT_BUILD_DEPENDENCIES = " \
    ament-lint-auto \
    ament-cmake-auto \
    ament-cmake-core \
    ament-cmake-cppcheck \
    ament-cmake-cpplint \
    ament-cmake-export-definitions \
    ament-cmake-export-dependencies \
    ament-cmake-export-include-directories \
    ament-cmake-export-interfaces \
    ament-cmake-export-libraries \
    ament-cmake-export-link-flags \
    ament-cmake-export-targets \
    ament-cmake-gmock \
    ament-cmake-gtest \
    ament-cmake-include-directories \
    ament-cmake-libraries \
    ament-cmake \
    ament-cmake-pytest \
    ament-cmake-python \
    ament-cmake-ros \
    ament-cmake-target-dependencies \
    ament-cmake-test \
    ament-cmake-version \
    ament-cmake-uncrustify \
    ament-cmake-flake8 \
    ament-cmake-pep257 \
    ament-copyright \
    ament-cpplint \
    ament-flake8 \
    ament-index-python \
    ament-lint-cmake \
    ament-mypy \
    ament-package \
    ament-pclint \
    ament-pep257 \
    ament-pycodestyle \
    ament-pyflakes \
    ament-uncrustify \
    ament-xmllint \
    cmake \
    eigen3-cmake-module \
    fastcdr \
    fastrtps-cmake-module \
    fastrtps \
    git \
    gmock-vendor \
    gtest-vendor \
    pkgconfig \
    python-cmake-module \
    python3-catkin-pkg \
    python3-empy \
    python3 \
    python3-nose \
    python3-pytest \
    rcutils \
    rmw-implementation-cmake \
    rosidl-cmake \
    rosidl-default-generators \
    rosidl-generator-c \
    rosidl-generator-cpp \
    rosidl-generator-dds-idl \
    rosidl-generator-py \
    rosidl-parser \
    rosidl-runtime-c \
    rosidl-runtime-cpp \
    rosidl-typesupport-c \
    rosidl-typesupport-cpp \
    rosidl-typesupport-fastrtps-cpp \
    rosidl-typesupport-interface \
    rosidl-typesupport-introspection-c \
    rosidl-typesupport-introspection-cpp \
    foonathan-memory-vendor \
    libyaml-vendor \
"

IMAGE_INSTALL:append = " \
    ros-base \
    examples-rclcpp-minimal-action-client \
    examples-rclcpp-minimal-action-server \
    examples-rclcpp-minimal-client \
    examples-rclcpp-minimal-composition \
    examples-rclcpp-minimal-publisher \
    examples-rclcpp-minimal-service \
    examples-rclcpp-minimal-subscriber \
    examples-rclcpp-minimal-timer \
    examples-rclcpp-multithreaded-executor \
    examples-rclpy-executors \
    examples-rclpy-minimal-action-client \
    examples-rclpy-minimal-action-server \
    examples-rclpy-minimal-client \
    examples-rclpy-minimal-publisher \
    examples-rclpy-minimal-service \
    examples-rclpy-minimal-subscriber \
    demo-nodes-cpp \
    demo-nodes-cpp-rosnative \
    demo-nodes-py \
    cyclonedds \
    rmw-cyclonedds-cpp \
    tmux \
    byobu \
    python3-argcomplete \
    glibc-utils \
    localedef \
    rt-tests \
    stress \
    xrt-dev \
    xrt \
    zocl \
    opencl-headers-dev \
    opencl-clhpp-dev \
    ${ROS_SYSROOT_BUILD_DEPENDENCIES} \
"

IMAGE_LINGUAS = "en-us"
GLIBC_GENERATE_LOCALES = "en_US.UTF-8"

EOF
```

---

#### ISSUE: 出現`git pull`授權失敗錯誤

###### step 1: 確認憑證是否存在

```sh
$ls /usr/local/oe-sdk-hardcoded-buildpath/sysroots/x86_64-petalinux-linux/etc/ssl/certs/
```

###### step 2: 創建憑證存放路徑

```sh
$sudo mkdir -p /usr/local/oe-sdk-hardcoded-buildpath/sysroots/x86_64-petalinux-linux/etc/ssl/certs/
```

---

###### step 3: 重新安裝憑證

```sh
$sudo rm -f /etc/ssl/certs/ca-bundle.crt
$sudo apt reinstall ca-certificates
$sudo update-ca-certificates
```

###### step 4: 複製憑證到指定路徑

```sh
$sudo cp /etc/ssl/certs/ca-certificates.crt
 /usr/local/oe-sdk-hardcoded-buildpath/sysroots/x86_64-petalinux-linux/etc/ssl/certs/
```

---

### 3-2. Build the Project

###### Before building the last thing you need to do is to add in project-spec/meta-user/conf/petalinuxbsp.conf the next line: (This solves some problems with building the image)

```sh
$vim project-spec/meta-user/conf/petalinuxbsp.conf
# append the following line
SIGGEN_UNLOCKED_RECIPES += "gcc-cross-aarch64"
```

```sh
$cd /krs_ws/<PLNX_PROJ_DIR>
# eg. cd /krs_ws/plnx_os
$petalinux-build
```

---

### 3-3. Package KR260 Boot Image

- 創建 kr260 開機映像檔

###### 預設在`images/linux`路徑下產生 sd-card 映像檔(petalinux-sdimage.wic)

```sh
$petalinux-package --wic
 --bootfiles "ramdisk.cpio.gz.u-boot boot.scr Image system.dtb"
# correct
$petalinux-package --wic
 --bootfiles "rootfs.cpio.gz.u-boot boot.scr Image system.dtb"
# update
$petalinux-package --wic
 --images-dir images/linux/
 --bootfiles "rootfs.cpio.gz.u-boot,boot.scr,Image,system.dtb,system-zynqmp-sck-kr-g-revB.dtb"
## or
$petalinux-package --wic
 --bootfiles "rootfs.cpio.gz.u-boot boot.scr Image system.dtb system-zynqmp-sck-kr-g-revB.dtb"
```

---

## 4. Verification

- 運行 ROS2 minimal publisher/subscriber 範例

```sh
$source /usr/bin/ros_setup.bash
$ros2 run examples_rclcpp_minimal_publisher publisher_lambda
```
