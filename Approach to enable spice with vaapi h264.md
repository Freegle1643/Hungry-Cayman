# Approach to enable spice with vaapi h264

## 1. Platform

Currently it has been test on the following platforms

**Software on Host Server**

- Host OS: Ubuntu 16.04x64

**Hardware on Host Server**

- Broadwell-U

**Software on Remote Client**

- Host OS: Ubuntu 16.04x64

**Hardware on Remote Client**

- Skylake

## 2. Summary

SPICE: Simple Protocol for Independent Computing Environments

SPICE is a remote display system built for virtual environments which allows you to view a computing 'desktop' environment not only on the machine where it is running, but from anywhere on the Internet and from a wide variety of machine architectures.

|      Software      | Description                              |
| :----------------: | ---------------------------------------- |
|      x11spice      | A tool that grabs framebuffer and send the content to spice server. |
|       spice        | Spice server                             |
|   spice-protocol   | Spice protocol                           |
|     spice-gtk      | Client library using glib and gtk, binary named "spicy" will be built out |
|     gstreamer      | GStreamer open-source multimedia framework core library |
|  gst-plugins-base  | 'Base' GStreamer plugins and helper libraries |
|  gst-plugins-good  | 'Good' GStreamer plugins                 |
|  gst-plugins-bad   | 'Bad' GStreamer plugins and helper libraries |
|  gst-plugins-ugly  | 'Ugly' GStreamer plugins                 |
|  gstreamer-vaapi   | Hardware-accelerated video decoding, encoding and processing on Intel graphics through VA-API |
|       libva        | Libva is an implementation for VA-API (Video Acceleration API) |
| intel-vaapi-driver | VA-API (Video Acceleration API) user mode driver for Intel GEN Graphics family |
|    libva-utils     | libva-utils is a collection of utilities and examples to exercise VA-API in accordance with the libva project. binary named "vainfo" will be built |

## 	3. Setup the common parts of Spice Server and Client

Following steps in this section are required for both host server and remote client as a precondition. During the installation process, you may encounter errors and the output would indicate some dependent packages are missing. Simply install the required package and remember run the configure script AGAIN after you fix those problems.

### 3.0 Setup the environment

Run the commands below to setup the directory for download source

```
 # mkdir -p /home/spice-h264
 # mkdir -p /home/spice-h264/resource
```

Save the script listed below as "env.sh" and place it in the root directory of project, i.e. `/home/spice-h264`.

```
 #! /bin/bash  
 export PATH=/home/spice-h264/resource/bin/:$PATH 
 export LD_LIBRARY_PATH=/home/spice-h264/resource/lib/:$LD_LIBRARY_PATH 
 export PKG_CONFIG_PATH=/home/spice-h264/resource/lib/pkgconfig/:/home/spice-h264/resource/share/pkgconfig:$PKG_CONFIG_PATH
```

Run the commands below before you start doing other things

```
 # cd /home/spice-h264
 # source env.sh
```

### 3.1 Install necessary dependencies

Because some of the installations require necessary dependencies like software or libraries to be successfully installed. Here list all the necessary dependencies I installed through my setup process in this section. You may or may not need to install all of them due to your setup on your specific machine, and this may not cover all the dependencies required. Please be informed you should install other extra dependencies if your set up fails with error output. Normally, based on the output, you are to find what you have to do to fix the problem accordingly. 

```bash
sudo apt-get install \
	autoconf automake libtool autopoint \
	gtk-doc* \
    glib* \
    gobject-introspection \
    libglib2.0-dev \
    libqt5gstreamer-1.0-0 libqtgstreamer-1.0-0 gir1.2-gstreamer-1.0 \
    libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \
    libxrandr* 
```

### 3.2 Build libva

Run the commands below to build libva

```
 # cd /home/spice-h264
 # git clone https://github.com/01org/libva.git
 # cd libva
 # git checkout v2.1-branch
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/resource/
 # make && make install
```

### 3.3 Build intel-vaapi-driver

Run the commands below to build intel-vaapi-driver

```
 # cd /home/spice-h264
 # git clone https://github.com/01org/intel-vaapi-driver.git
 # cd intel-vaapi-driver
 # git checkout v2.1-branch
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/resource/
 # make && make install
```

### 3.4 Build libva-utils

Run the commands below to build libva-utils

```
 # cd /home/spice-h264
 # git clone https://github.com/intel/libva-utils
 # cd libva-utils
 # git checkout v2.1-branch
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/resource/
 # make && make install
```

### 3.5 Build gstreamer

Run the commands below to build gstreamer.

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gstreamer.git
 # cd gstreamer
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/resource/
 # make && make install
```

### 3.6 Build gst-plugins-base

Run the commands below to build gst-plugins-base

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gst-plugins-base.git
 # cd gst-plugins-base
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure LDFLAGS="-L/home/spice-h264/gst-plugins-base/gst-libs/gst/video/.libs/" --prefix=/home/spice-h264/resource/
 # make && make install
```

THE `LDFLAGS` is used to fix the error `/bin/ld: cannot find -lgstvideo-1.0` when building `gst-plugins-base/tests/examples/gl/sdl`

### 3.7 Build gst-plugins-good

Run the commands below to build gst-plugins-good

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gst-plugins-good.git
 # cd gst-plugins-good
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/resource/
 # make && make install
```

### 3.8 Build gst-plugins-bad

Run the commands below to build gst-plugins-bad

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gst-plugins-bad.git
 # cd gst-plugins-bad
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/resource/
 # make && make install
```

### 3.9 Build gst-plugins-ugly

Run the commands below to build gst-plugins-ugly

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/gstreamer/gst-plugins-ugly.git
 # cd gst-plugins-ugly
 # git checkout 1.13.1
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/resource/
 # make && make install
```

### 3.10 Build gstreamer-vaapi

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
 # ./configure --prefix=/home/spice-h264/resource/
 # make && make install
```

### 3.11 Build spice-protocol

Run the commands below to build spice-protocol

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/spice/spice-protocol.git
 # cd spice-protocol
 # git checkout v0.12.13
 # ./autogen.sh
 # ./configure --prefix=/home/spice-h264/resource/
 # make && make install
```

## 4. Setup Spice Server

Following steps in this section are required for host server, which spice server will be running on. During the installation process, you may encounter errors and the output would indicate some dependent packages are missing. Simply install the required package and remember run the configure script AGAIN after you fix those problems.

DO NOT forget to check whether the environment is set correctly: `/home/spice-h264/resource/` should be set in `$PATH`, `$LD_LIBRARY_PATH` and `$PKG_CONFIG_PATH`.
If NOT, run the commands below to fix them.

```
 # cd /home/spice-h264
 # source env.sh
```

### 4.0 Install necessary dependencies 

Because some of the installations require necessary dependencies like software or libraries to be successfully installed. Here list all the necessary dependencies I installed through my setup process in this section. You may or may not need to install all of them due to your setup on your specific machine, and this may not cover all the dependencies required. Please be informed you should install other extra dependencies if your set up fails with error output. Normally, based on the output, you are to find what you have to do to fix the problem accordingly. 

```bash
sudo apt-get install \
	liborc-0.4-* \
	openssl \
	libssl-dev \
	libjpeg-dev \
	libxcb-damage0* libxcb-xtest0* libxcb-xkb* \
	gir1.2-gtk-3.0 libgtk-3-0 libgtk-3-0-dbg gir1.2-spice-client-gtk-3.0 libspice-client-gtk-3.0-4 libspice-client-gtk-3.0-dev 
```

### 4.1 Build spice

Save the patch below and apply the patch before building spice server

` spice.patch` as attached in the email or package you received.

Run the commands below to build spice server

```
 # cd /home/spice-h264
 # git clone  https://anongit.freedesktop.org/git/spice/spice.git
 # cd spice
 # git checkout 0.14
 # ./autogen.sh
 # ./configure --disable-celt051 --prefix=/home/spice-h264/resource/
 # make && make install
```

During the building process, following warning message may be printed out, if so, make sure x264enc is enabled in gst-plugins-ugly, it can be checked during the configure step of gst-plugins-ugly

```
 configure: WARNING: The x264enc GStreamer element(s) are missing. You should be able to find them in the gst-plugins-ugly 1.0 package.
```

CELT is a very low delay audio codec designed for high-quality communications. If you want to enable it, download the source code from link below and remove `--disable-celt051` from configure option

```
 # cd /home/spice-h264
 # wget https://src.fedoraproject.org/repo/pkgs/celt051/celt-0.5.1.3.tar.gz/67e7b5e45db57a6f1f0a6962f5ecb190/celt-0.5.1.3.tar.gz
 # tar xzf celt-0.5.1.3.tar.gz
 # cd celt-0.5.1.3
 # ./configure --prefix=/home/spice-h264/resource/
 # make && make install
```

### 4.2 Build x11spice

Save the patch below and apply the patch before building x11spice

` x11spice.patch` as attached in the email or package you received.

Run the commands below to build x11spice

```
 # cd /home/spice-h264
 # git clone https://github.com/jwhite66/x11spice.git
 # cd x11spice
 # ./autogen.sh
 # ./configure CFLAGS="-Wl,--no-as-needed -Wno-error" --prefix=/home/spice-h264/resource/
 # make
```

### 4.3 Test Spice Server

x11spice will grab the framebuffer on host server and send the framebuffer to remote client by calling spice server API.
Run the commands below to run x11spice.

```
 # cd /home/spice-h264/x11spice
 # ./src/x11spice --password=1 --allow-control
```

*Note that you can modify the `--password` option to any password you prefer*

## 5. Setup the Spice Client

Following steps in this section are required for remote client, which spice client will be running on. During the installation process, you may encounter errors and the output would indicate some dependent packages are missing. Simply install the required package and remember run the configure script AGAIN after you fix those problems.

DO NOT forget to check whether the environment is set correctly: `/home/spice-h264/resource/` should be set in `$PATH`, `$LD_LIBRARY_PATH` and `$PKG_CONFIG_PATH`.
If NOT, run the commands below to fix them.

```
 # cd /home/spice-h264
 # source env.sh
```

### 5.0 Install necessary dependencies

Because some of the installations require necessary dependencies like software or libraries to be successfully installed. Here list all the necessary dependencies I installed through my setup process in this section. You may or may not need to install all of them due to your setup on your specific machine, and this may not cover all the dependencies required. Please be informed you should install other extra dependencies if your set up fails with error output. Normally, based on the output, you are to find what you have to do to fix the problem accordingly. 

```bash
sudo apt-get install \
	valac* \
	debhelper  libpixman-1-dev  libssl-dev  libgtk-3-dev   libglib2.0-dev  libgudev-1.0-dev  libcairo2-dev  libpulse-dev   libusb-1.0-0-dev \ 
	python-all   python-six   python-gtk2-dev  python-pyparsing \ 
	libsasl2-dev  libjpeg-dev \ 
	gobject-introspection \ 
	libgirepository1.0-dev gir1.2-gtk-2.0 libtext-csv-perl   libusbredirhost-dev  libacl1-dev  libpolkit-agent-1-dev libpolkit-gobject-1-dev \ 
	dpkg-dev  libdbus-glib-1-dev libopus-dev  libsoup2.4-dev \ 
	gtk-doc-tools liblz4-dev  libcacard-dev  libspice-protocol-dev   libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gettext libepoxy-dev 
```



### 5.1 Build spice-gtk

Save the patch below and apply the patch before building spice client

`spice-gtk.patch` as attached in the email or package you received.

Run the commands below to build spice client

```
 # cd /home/spice-h264
 # git clone https://anongit.freedesktop.org/git/spice/spice-gtk.git
 # git checkout v0.34
 # ./autogen.sh
 # ./configure --enable-vala --prefix=/home/spice-h264/resource/
 # make && make install
```

### 5.2 Test Spice Client

Run the commands below to run spice client.

```
 # cd /home/spice-h264/spice-gtk
 # ./tools/spicy
```

Fill the hostname with host server's IP and fill the port with `5900`, then click `connect` button to build connection with spice server.

If the `gstreamer-vaapi.patch` is applied before building gstreamer-vaapi, then following messages can be seen:

On host server side

```
 Enter into file: gstvaapiencoder_h264.c, func: gst_vaapi_encoder_h264_encode, line: 2911
 Enter into file: gstvaapiencoder_h264.c, func: gst_vaapi_encoder_h264_encode, line: 2911
 Enter into file: gstvaapiencoder_h264.c, func: gst_vaapi_encoder_h264_encode, line: 2911
 ...... 
```

On remote client side

```
 Enter into file: gstvaapidecoder_h264.c, func: gst_vaapi_decoder_h264_decode, line: 4728
 Enter into file: gstvaapidecoder_h264.c, func: gst_vaapi_decoder_h264_decode, line: 4728
 Enter into file: gstvaapidecoder_h264.c, func: gst_vaapi_decoder_h264_decode, line: 4728
 ......
```

