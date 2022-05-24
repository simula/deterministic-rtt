# Network Path Integrity Verification using Deterministic Delay Measurements

## Build description for custom real time kernel on RPI4 ##



### Usable Cmake and build instructions
We only need the [Cmake](2021-12-28-Building-a-Linux-kernel-with-OSD.txt) recipe which automate the installation of a fully functional Yocto build environment and the custom Yocto build recipe for our Linux realtime kernel. You can follow instructions from this file on continue the the rest of the readme.

### Build Instructions ###
#### Clone the project ####
The image is based on Yocto recipes, and we use CMake from the publicly available repository. The following will check out a read-only version of OSD (Open Source Distribution), a CMake-based package manager for open source software by Stein J Ryan.
```
mkdir ~/work
cd ~/work
svn co https://prima.svn.beanstalkapp.com/osd/trunk osd
``` 

#### Update Cmake file ####
To use OSD to build a minimalistic Linux realtime kernel for the RaspberryPi4, suitable as a ping target with preferred link level address `169.254.209.123` create a file `~/work/CMakeLists.txt` with the following content:
```
project(superbuild)
cmake_minimum_required(VERSION 3.9.3)
include(ExternalProject)
add_subdirectory (osd/yocto/dunfell) #dunfell (nov21) or zeus (apr20)
add_subdirectory (osd/yocto-pinger/trunk)
```

#### Create necessary folders structure and create Yocto configuration  ####
Then continue as follows to generate a directory structure containing a bitbake configuration file for use with Yocto:
```
mkdir ~/work/local
mkdir ~/work/build.local
cd ~/work/build.local
cmake -DCMAKE_INSTALL_PREFIX=~/work/local ~/work
cmake --build .
```
The build will proceed to set up a Yocto configuration directory
```
...
[100%] Completed 'yocto-pinger'
[100%] Built target yocto-pinger
```
You now have a Yocto configuration directory under `~/work/local/yocto/meta-pinger/conf` which we will feed to the Yocto init script in order to create a build environment further below.

#### Build the image ####
First, we need to install all required Linux command-line tools. Please note that this list of tools might not be complete, and this could cause some of the bitbake recipes to fail in the middle of the build. The following may be sufficient to install all the required tools:
```
sudo apt-get install git chrpath diffstat texinfo
```

Now open up a `bash` command prompt (bash is required for the "source" command):

```
bash
cd ~/work/local/yocto/poky
TEMPLATECONF=~/work/local/yocto/meta-pinger/conf source ./oe-init-build-env ~/work/local/yocto/pinger.build
```
You can now proceed to start the actual build process. If you machine has limited memory (< 16 GB), use the following command to start the build in order to limit the use of memory 
```BB_NUMBER_THREADS=1 bitbake core-image-minimal
``` 
otherwise use the following command to employ all available CPU cores:
```
bitbake core-image-minimal
```
Which will start off like this and then exercise your hard drive for hours while building the Linux kernel and userland via the Yocto bitbake recipes
```
Build Configuration:
BB_VERSION           = "1.44.0"
BUILD_SYS            = "x86_64-linux"
NATIVELSBSTRING      = "universal"
TARGET_SYS           = "arm-osd-linux-gnueabi"
MACHINE              = "raspberrypi4"
DISTRO               = "osd"
DISTRO_VERSION       = "1.0"
TUNE_FEATURES        = "arm vfp cortexa7 neon vfpv4 thumb callconvention-hard"
TARGET_FPU           = "hard"
meta                 
meta-poky            
meta-yocto-bsp       
meta-initramfs       
meta-oe              
meta-python          
meta-networking      
meta-webserver       
meta-raspberrypi     
meta-pinger        = "<unknown>:<unknown>"
```
Umpteen hours later, the result is available in the form of an SD image file: `~/work/local/yocto/pinger.build/tmp/deploy/images/raspberrypi4/core-image-minimal-raspberrypi4.rpi-sdimg` which can be written to an SD card. It contains a "U-boot" boot monitor which receives control and starts the Linux kernel.


### Flash image to sd-card ###
For details on how to flash an SD card: https://www.raspberrypi.org/documentation/installation/installing-images/linux.md

For example:
```
lsblk -p
/dev/sdb1
sudo umount /dev/sdb1
sudo dd bs=4M if=~/work/local/yocto/pinger.build/tmp/deploy/images/raspberrypi4/core-image-minimal-raspberrypi4.rpi-sdimg of=/dev/sdb status=progress conv=fsync
125829120 bytes (126 MB, 120 MiB) copied, 17,8409 s, 7,1 MB/s
sync
```

### Link-local address verification ###
We can check the link-local address by connecting via serial console:
```
sudo screen /dev/ttyUSB0 115200

/ # udhcpc
udhpc: started, v1.31.0
[   43.749021] bcmgenet: Skipping UMAC reset
[   43.864774] bcmgenet fd580000.genet: configuring instance for external RGMII (no delay)
udhcpc: sending discover
[   44.884125] bcmgenet fd580000.genet eth0: Link is Down
udhcpc: sending discover
[   49.044289] bcmgenet fd580000.genet eth0: Link is Up - 1Gbps/Full - flow control off
udhcpc: sending discover
udhcpc: sending discover
udhcpc: sending discover
udhcpc: sending select for 192.168.240.160
udhcpc: lease of 192.168.240.160 obtained, lease time 604800
/etc/udhcpc.d/50default: Adding DNS 192.168.240.1
/ # uname -v
#1 SMP PREEMPT RT Fri Aug 21 10:11:31 UTC 2021
```



