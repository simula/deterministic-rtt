# Network Path Integrity Verification using Deterministic Delay Measurements

## Build description for custom real time kernel on RPI4 ##

### Clone the project ###
The image is based on Yocto recipes, and we use CMake from the publicly available repository
```
svn co https://prima.svn.beanstalkapp.com/osd/trunk osd
```

### Usable Cmake
We only need the [Cmake](2021-12-28-Building-a-Linux-kernel-with-OSD.txt) recipe which automate the installation of a fully functional Yocto build environment and the custom Yocto build recipe for our Linux realtime kernel.

