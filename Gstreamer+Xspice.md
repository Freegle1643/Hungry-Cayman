# Gstreamer + Xspice in Docker Container

## 1. Initiation and Abstract

Since the approach to build Xspice from scratch inside container failed with multiple mysterious errors, I decided to simply use `xserver-xspice` package as our final approach to *headlessly* launch SPICE inside container for remote desktop connection. 

I have carefully inspect Xspice project, which is currently part of [`xf86-video-qxl`](https://cgit.freedesktop.org/xorg/driver/xf86-video-qxl) project. I researched the source code and discover Xspice links and calls gstreamer automatically when gstreamer is set up correctly. Also, all the parameters to adjust the performance of gstreamer could be parse as parameters to launch Xspice, or one can simply use a provided configure file to make changes accordingly. 

So for now, my plan is to set up gstreamer inside Docker container that takes advantages of Intel graphic card to do dodec of video streaming. Then use Xspice to setup a headless remote control environment for users to connect. User may also try many different apps or desktops as well.

## 2. Setup Gstreamer in a Docker container

 Let's try to start a basic Ubuntu container and install all the necessary package. Because previous successfully trial was that I mounted using `-v` to have the native host Linux directory mounted on. So let's try pure container and find out what's what.

### Start container

```bash
docker run -it \
	-e DISPLAY=$:3.0 \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	--device /dev/dri \
	--device /dev/snd \
	-p 5997:5900 ubuntu:16.04
```

### 2.0 Setup the environment

Run the commands below to setup the directory for download source

```bash
 mkdir -p /home/spice-h264
 mkdir -p /home/spice-h264/src
```

Save the script listed below as "env.sh" and place it in the root directory of project, i.e. `/home/spice-h264`.

```shell
#! /bin/bash  
export PATH=/home/spice-h264/src/bin/:$PATH 
export LD_LIBRARY_PATH=/home/spice-h264/src/lib/:$LD_LIBRARY_PATH 
export PKG_CONFIG_PATH=/home/spice-h264/src/lib/pkgconfig/:/home/spice-h264/src/share/pkgconfig:$PKG_CONFIG_PATH
```

Run the commands below before you start doing other things

```bash
 cd /home/spice-h264
 source env.sh
```

### 2.1 Install necessary dependencies

Because some of the installations require necessary dependencies like software or libraries to be successfully installed. Here list all the necessary dependencies I installed through my setup process in this section. You may or may not need to install all of them due to your setup on your specific machine, and this may not cover all the dependencies required. Please be informed you should install other extra dependencies if your set up fails with error output. Normally, based on the output, you are to find what you have to do to fix the problem accordingly. 

```bash
sudo apt-get install \
	autoconf automake libtool autopoint \
	gtk-doc* glib* gobject-introspection \
    libglib2.0-dev \
    gir1.2-gstreamer-1.0 \
    libxrandr* \
    libdrm-dev libdrm-intel1 \
    bison flex 
```

### 2.2 Build libva

Run the commands below to build libva

```
 # cd /home/spice-h264
 # git clone https://github.com/01org/libva.git
 # cd libva
 # git checkout v2.1-branch
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/src/
 # make && make install
```

### 2.3 Build intel-vaapi-driver

Run the commands below to build intel-vaapi-driver

```
 # cd /home/spice-h264
 # git clone https://github.com/01org/intel-vaapi-driver.git
 # cd intel-vaapi-driver
 # git checkout v2.1-branch
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/src/
 # make && make install
```

### 2.4 Build libva-utils

Run the commands below to build libva-utils

```
 # cd /home/spice-h264
 # git clone https://github.com/intel/libva-utils
 # cd libva-utils
 # git checkout v2.1-branch
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/src/
 # make && make install
```

### 2.5 Build gstreamer

Run the commands below to build gstreamer.

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gstreamer.git
 # cd gstreamer
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/src/
 # make && make install
```

### 2.6 Build gst-plugins-base

Run the commands below to build gst-plugins-base

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gst-plugins-base.git
 # cd gst-plugins-base
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure LDFLAGS="-L/home/spice-h264/gst-plugins-base/gst-libs/gst/video/.libs/" --prefix=/home/spice-h264/src/
 # make && make install
```

THE `LDFLAGS` is used to fix the error `/bin/ld: cannot find -lgstvideo-1.0` when building `gst-plugins-base/tests/examples/gl/sdl`

### 2.7.0 Build Orc

When makeing gst-plugins-good, I discover that this may accelerate the overall performance

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/orc.git
 # cd orc
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/src/
 # make && make install
```

 

### 2.7 Build gst-plugins-good

Run the commands below to build gst-plugins-good

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gst-plugins-good.git
 # cd gst-plugins-good
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/src/
 # make && make install
```

### 2.8 Build gst-plugins-bad

Run the commands below to build gst-plugins-bad

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gst-plugins-bad.git
 # cd gst-plugins-bad
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/src/
 # make && make install
```

### 2.9 Build gst-plugins-ugly

Run the commands below to build gst-plugins-ugly

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gst-plugins-ugly.git
 # cd gst-plugins-ugly
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/src/
 # make && make install
```

### 2.10 Build gstreamer-vaapi

Save the patch below and apply the patch before building gstreamer-vaapi.

`gstreamer-vaapi.patch` as attached in the email or package you received.

This patch is NOT a must. However, it will print out whether VAAPI's encode/decode function is called or not.

Run the commands below to build gstreamer-vaapi

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gstreamer-vaapi.git
 # cd gstreamer-vaapi
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/src/
 # make && make install
```

### 2.11 Build spice-protocol

Run the commands below to build spice-protocol

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/spice/spice-protocol.git
 # cd spice-protocol
 # git checkout v0.12.13
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/src/
 # make && make install
```

## 