# Daily Log for Project Hungry Cayman

*Fiat lux*

## 03-14

### To-Dos

- 配置好 [XSpice](https://cgit.freedesktop.org/xorg/driver/xf86-video-qxl) 和 GStreamer 阅读代码的环境


- 找寻 GStreamer 和 XSpice 是如何关联的
- 对 GStreamer 的概要和使用有初步了解

### Logs

#### 在XSpice代码中找寻有关gstreamer相关的部分

```
Searching 77 files for "gstreamer"

D:\CWorkspace\xf86-video-qxl_origin\examples\spiceqxl.xorg.conf.example:
  102      # Sets a semicolon-separated list of preferred video codecs.
  103      # Each takes the form encoder:codec, with spice:mjpeg being the default,
  104:     # and other options being provided by gstreamer for the mjpeg, vp8 and h264
  105      # codecs.
  106      #Option "SpiceVideoCodecs" ""


D:\CWorkspace\xf86-video-qxl_origin\scripts\Xspice:
   98  parser.add_argument('--streaming-video', choices=['off', 'all', 'filter'],
   99                      help='set the streaming video method')
  100: parser.add_argument('--video-codecs', help='set a semicolon-separated list of preferred video codecs to use. Each takes the form encoder:codec, with spice:mjpeg being the default and other options being provided by gstreamer for the mjpeg, vp8 and h264 codecs')
  101  
  102  # VDAgent options


2 matches across 2 files

```

##### spiceqxl.xorg.conf.example

`examples\spiceqxl.xorg.conf.example`

即说明gstreamer是由spice直接调用，并直接使用gstreamer进行视频编解码的，所以理论上说我们需要做的就是沿着这条调用的路子（#Option "SpiceVideoCodecs"）去把XSpice和gstreamer的关系理清楚。

##### Xspice

`scripts\Xspice`

这是一个可以通过传参来再命令行运行的XSpice的解析模块，它会把用户在'--video-codecs'部分传递的参数解析，进而对gstreamer进行相应的调用。

### Left

根据gstreamer官网的指示，等后面去Linux里看一看是怎么回事

> Linux
>
> Most, if not all, Linux distributions provide packages of GStreamer. You should find these in your distribution's package repository. 
> Note that some distributions split the GStreamer plugins up further than the upstream sources. Additionally, some distributions do not include the gst-plugins-bad, gst-plugins-ugly, and gst-libav packages in their main repository, for legal reasons.



## 3-20

### To-Dos

- Review SPICE with Vaapi h.264 Doc
  - Document literacy
  - Step-by-step correctness

### Logs

- 文檔中libva的安裝出現了warning，已記截屏記錄
- 另外文檔要求兩臺可以互聯的Intel Linux，我其實并沒有這樣的條件，但是先配置吧，反正可以把環境先搭起來。
- gstreamer安裝時出現如下錯誤

```bash
root@haoyuan-Broadwell:/home/spice-h264/gstreamer# ./autogen.sh
+ check for build tools
  checking for autoreconf ... 
/usr/bin/autoreconf
  checking for pkg-config ... 
/usr/bin/pkg-config
+ checking for autogen.sh options
  This autogen script will automatically run ./configure as:
  ./configure --enable-maintainer-mode --enable-gtk-doc --enable-failing-tests --enable-poisoning
  To pass any additional options, please specify them on the ./autogen.sh
  command line.
+ running autopoint --force...
./autogen.sh: 108: ./autogen.sh: autopoint: not found

autopoint failed
```


### Left

 从gstreamer往后的所有安装配置工作

## 3-21

### To-Dos

- 尽量配置完所有的步骤

### Logs

- 解决3-20的`autopoint failed`错误：

【!】根据网上查询的资料，需要使用如下命令先安装一些应用程序

```bash
apt-get install autoconf automake libtool autopoint
```

之后再次运行`./autogen.sh`还是出现问题：

```bash
checking for valgrind... no
checking for gobject-introspection... no
checking for gtkdoc-check... no
checking for gtkdoc-rebase... no
checking for gtkdoc-mkpdf... no
configure: error: You need to have gtk-doc >= 1.12 installed to build GStreamer
  configure failed
```

【!】根据`apt-cache search`的搜寻， 又安装了`gtk-doc*`，再次运行出现下列报错：

```bash
checking for valgrind... no
checking for gobject-introspection... no
checking for gtkdoc-check... /usr/bin/gtkdoc-check
checking for gtkdoc-rebase... /usr/bin/gtkdoc-rebase
checking for gtkdoc-mkpdf... /usr/bin/gtkdoc-mkpdf
checking for GTKDOC_DEPS... configure: error: Package requirements (glib-2.0 >= 2.10.0 gobject-2.0  >= 2.10.0) were not met:

No package 'glib-2.0' found
No package 'gobject-2.0' found

Consider adjusting the PKG_CONFIG_PATH environment variable if you
installed software in a non-standard prefix.

Alternatively, you may set the environment variables GTKDOC_DEPS_CFLAGS
and GTKDOC_DEPS_LIBS to avoid the need to call pkg-config.
See the pkg-config man page for more details.

  configure failed
```

{

根据`apt-cache search`的搜寻， 预计安装如下两个包：

```bash
gir1.2-glib-2.0 - Introspection data for GLib, GObject, Gio and GModule
gir1.2-spice-client-glib-2.0 - GObject for communicating with Spice servers (GObject-Introspection)
```

Sadly，还是同样的报错，于是从valgrind开始逐个安装

```bash
apt get install vavlgrind*
```

成功安装`checking for valgrind... /usr/bin/valgrind`

根据`apt-cache search`的搜寻`gobject-introspection`，相关的包很多，悉数安装

```bash
apt-get install gir1.2-gconf-2.0 gir1.2-gdkpixbuf-2.0 gir1.2-gee-0.8 gir1.2-gudev-1.0 gir1.2-secret-1 gir1.2-telepathyglib-0.12 gobject-introspection  gir1.2-farstream-0.1  gir1.2-farstream-0.2  gir1.2-folks-0.6   gir1.2-libvirt-glib-1.0   gir1.2-farstream-0.2   gir1.2-folks-0.6   gir1.2-libvirt-glib-1.0   gir1.2-spice-client-glib-2.0  gir1.2-spice-client-gtk-2.0  gir1.2-spice-client-gtk-3.0   gir1.2-uhm-0.0
```

仍然有错

> Xinda: no不用管，关键还是那些missing的package

}

于是上面{}内的暂且不用管了，我们根据

```bash
checking for GTKDOC_DEPS... configure: error: Package requirements (glib-2.0 >= 2.10.0 gobject-2.0  >= 2.10.0) were not met:

No package 'glib-2.0' found
No package 'gobject-2.0' found
```

【!】去安装glib和gobject：

```bash
apt-get install glib*
apt-get install gobject-introspection
```

如果再出错，根据网上的搜寻，再安装下面的软件包即可，至此，gstreamer安装完成

```bash
apt-get install libglib2.0-dev
```

成功信息

```bash
make[1]: Leaving directory '/home/spice-h264/gstreamer'
[1]+  Exit 2                  make
```

- 安装gst-plugins-base

`./autogen.sh` 出现如下错误 

```bash
configure: No package 'gstreamer-1.0' found
configure: error: no gstreamer-1.0 >= 1.13.1 (GStreamer) found
  configure failed
```

？？？那我上一步安的是什么东西？？？

我们先看如何解决

```bash
root@haoyuan-Broadwell:/home/spice-h264/gst-plugins-base# apt-cache search gstreamer-1.0
libqt5gstreamer-1.0-0 - C++ bindings library for GStreamer with a Qt-style API - Qt 5 build
libqtgstreamer-1.0-0 - C++ bindings library for GStreamer with a Qt-style API
gir1.2-gstreamer-1.0 - GObject introspection data for the GStreamer library
```

悉数安装

```bash
apt-get install libqt5gstreamer-1.0-0 libqtgstreamer-1.0-0 gir1.2-gstreamer-1.0
```

同样的问题，有搜集了网上的答案

```bash
apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
```

还是不行，暂时留到明天

### Left

- 解决gstreamer安装问题

## 3-23

### To-Dos

- 解决之前卡住的gst-plugins-base安装问题
- 力争过完所有的doc步骤
- 因为机器原因，仅在一台单机上尝试

### Logs

- gst-plugins-base

你知道最绝望的事是什么吗，一天没动，`autogen.sh`就可以通过了 ：）现在代码也流行风水和佛系了吗？？？

随缘。

记录一些会缺少的功能

```bash
configure: *** Plug-ins without external dependencies that will be built:
	adder
	app
	audioconvert
	audiomixer
	audiorate
	audioresample
	audiotestsrc
	encoding
	gio
	playback
	rawparse
	subparse
	tcp
	typefind
	videoconvert
	videorate
	videoscale
	videotestsrc
	volume

configure: *** Plug-ins without external dependencies that will NOT be built:

configure: *** Plug-ins that have NOT been ported:

configure: *** Plug-ins with dependencies that will be built:
	 
	gl
	ximagesink
	xvimagesink

configure: *** Plug-ins with dependencies that will NOT be built:
	alsa
	cdparanoia
	ivorbisdec
	libvisual
	ogg
	opus
	pango
	theora
	vorbis

configure: *** Orc acceleration disabled.  Requires Orc >= 0.4.24, which was
               not found.  Slower code paths will be used.

Now type 'make' to compile gst-plugins-base.
```

configure之后：

（感觉可能一些包还需要安装以提升性能）

```bash
configure: *** Plug-ins without external dependencies that will be built:
	adder
	app
	audioconvert
	audiomixer
	audiorate
	audioresample
	audiotestsrc
	encoding
	gio
	playback
	rawparse
	subparse
	tcp
	typefind
	videoconvert
	videorate
	videoscale
	videotestsrc
	volume

configure: *** Plug-ins without external dependencies that will NOT be built:

configure: *** Plug-ins that have NOT been ported:

configure: *** Plug-ins with dependencies that will be built:
	 
	gl
	ximagesink
	xvimagesink

configure: *** Plug-ins with dependencies that will NOT be built:
	alsa
	cdparanoia
	ivorbisdec
	libvisual
	ogg
	opus
	pango
	theora
	vorbis

configure: *** Orc acceleration disabled.  Requires Orc >= 0.4.24, which was
               not found.  Slower code paths will be used.
```

之后编译成功

- gst-plugins-good

configure后一些功能显示可能会缺少：

```bash
configure: *** Plug-ins without external dependencies that will be built:
	alpha
	apetag
	audiofx
	audioparsers
	auparse
	autodetect
	avi
	cutter
	debugutils
	deinterlace
	dtmf
	effectv
	equalizer
	flv
	flx
	goom
	goom2k1
	icydemux
	id3demux
	imagefreeze
	interleave
	isomp4
	law
	level
	matroska
	multifile
	multipart
	replaygain
	rtp
	rtpmanager
	rtsp
	shapewipe
	smpte
	spectrum
	udp
	videobox
	videocrop
	videofilter
	videomixer
	wavenc
	wavparse
	y4m

configure: *** Plug-ins without external dependencies that will NOT be built:
	monoscope

configure: *** Plug-ins that have NOT been ported:

configure: *** Plug-ins with dependencies that will be built:
	oss4
	ossaudio
	png
	video4linux2
	ximagesrc

configure: *** Plug-ins with dependencies that will NOT be built:
	1394
	aasink
	cacasink
	cairo
	directsoundsink
	dv
	flac
	gdkpixbuf
	gtk
	jack
	jpeg
	lame
	mpg123
	osxaudio
	osxvideosink
	pulseaudio
	qt
	shout2
	souphttpsrc
	speex
	taglib
	twolame
	vpx
	waveformsink
	wavpack

configure: *** Orc acceleration disabled.  Requires Orc >= 0.4.17, which was
               not found.  Slower code paths will be used.
```

编译安装通过

- gst-plugins-bad

configure后一些功能显示可能会缺少：

```bash
configure: *** Plug-ins without external dependencies that will be built:
        accurip
        adpcmdec
        adpcmenc
        aiff
        asfmux
        audiobuffersplit
        audiofxbad
        audiomixmatrix
        audiovisualizers
        autoconvert
        bayer
        camerabin2
        coloreffects
        compositor
        debugutils
        dvbsuboverlay
        dvdspu
        faceoverlay
        festival
        fieldanalysis
        freeverb
        frei0r
        gaudieffects
        gdp
        geometrictransform
        id3tag
        inter
        interlace
        ivfparse
        ivtc
        jp2kdecimator
        jpegformat
        librfb
        midi
        mpegdemux
        mpegpsmux
        mpegtsdemux
        mpegtsmux
        mxf
        netsim
        onvif
        pcapparse
        pnm
        proxy
        rawparse
        removesilence
        sdp
        segmentclip
        siren
        smooth
        speed
        stereo
        subenc
        timecode
        videofilters
        videoframe_audiolevel
        videoparsers
        videosignal
        vmnc
        y4m
        yadif

configure: *** Plug-ins without external dependencies that will NOT be built:

configure: *** Plug-ins that have NOT been ported:

configure: *** Plug-ins with dependencies that will be built:
        dash
        decklink
        dvb
        fbdevsink
        gl
        hls
        ipcpipeline
        kms
        shm
        smoothstreaming
        vcdsrc

configure: *** Plug-ins with dependencies that will NOT be built:
        acm
        androidmedia
        aom
        applemedia
        assrender
        avcsrc
        bluez
        bs2b
        bz2
        chromaprint
        curl
        daala
        dc1394
        dfbvideosink
        direct3dsink
        directsoundsrc
        dtls
        dtsdec
        faac
        faad
        fdkaac
        flite
        fluidsynth
        gme
        gsmenc gsmdec
        iqa
        kate
        ladspa
        lcms2
        libde265
        libmms
        lv2
        modplug
        mpeg2enc
        mplex
        msdk
        musepack
        neonhttpsrc
        nvdec
        nvenc
        ofa
        openal
        opencv
        openexr
        openh264
        openjpeg
        openmpt
        openni2
        opensl
        opus
        resindvd
        rsvg
        rtmp
        sbc
        schro
        sfdec sfenc
        soundtouch
        spandsp
        spc
        srt
        srtp
        teletextdec
        tinyalsa
        ttml
        uvch264
        vdpau
        vo-aacenc
        vo-amrwbenc
        vulkan
        wasapi
        wayland
        webp
        webrtc
        webrtcdsp
        wildmidi
        winks
        winscreencap
        x265
        zbar

configure: *** Orc acceleration disabled.  Requires Orc >= 0.4.17, which was
               not found.  Slower code paths will be used.
```



- gst-plugins-ugly

configure后一些功能显示可能会缺少：

```bash
configure: *** Plug-ins without external dependencies that will be built:
	asfdemux
	dvdlpcmdec
	dvdsub
	realmedia
	xingmux

configure: *** Plug-ins without external dependencies that will NOT be built:

configure: *** Plug-ins that have NOT been ported:

configure: *** Plug-ins with dependencies that will be built:

configure: *** Plug-ins with dependencies that will NOT be built:
	a52dec
	amrnb
	amrwbdec
	cdio
	dvdreadsrc
	mpeg2dec
	sid
	x264

configure: *** Orc acceleration disabled.  Requires Orc >= 0.4.16, which was
               not found.  Slower code paths will be used.

```

编译安装通过

- gstreamer-vaapi

`./autogen.sh`时出错：

```bash
configure: No package 'gstreamer-codecparsers-1.0' found
configure: error: no gstreamer-codecparsers-1.0 >= 1.13.1 (yes) found
  configure failed
```

出错是没有安bad，忘记了

configure后一些功能显示可能会缺少：

```bash

gstreamer-vaapi configuration summary:

Installation Prefix .............. : /home/spice-h264/resource
GStreamer API version ............ : 1.13
VA-API version ................... : 1.1.0
Video encoding ................... : yes
Video outputs .................... : drm x11 glx egl wayland

```

make失败：

```bash
gstvaapidisplay_x11.c:39:36: fatal error: X11/extensions/Xrandr.h: No such file or directory
compilation terminated.
Makefile:1372: recipe for target 'libgstvaapi_x11_la-gstvaapidisplay_x11.lo' failed
make[4]: *** [libgstvaapi_x11_la-gstvaapidisplay_x11.lo] Error 1
make[4]: Leaving directory '/home/spice-h264/gstreamer-vaapi/gst-libs/gst/vaapi'
Makefile:471: recipe for target 'all-recursive' failed
make[3]: *** [all-recursive] Error 1
make[3]: Leaving directory '/home/spice-h264/gstreamer-vaapi/gst-libs/gst'
Makefile:471: recipe for target 'all-recursive' failed
make[2]: *** [all-recursive] Error 1
make[2]: Leaving directory '/home/spice-h264/gstreamer-vaapi/gst-libs'
Makefile:542: recipe for target 'all-recursive' failed
make[1]: *** [all-recursive] Error 1
make[1]: Leaving directory '/home/spice-h264/gstreamer-vaapi'
Makefile:474: recipe for target 'all' failed
make: *** [all] Error 2
```

根据提示：

```bash
root@haoyuan-Broadwell:/home/spice-h264/gstreamer-vaapi# apt-cache search Xrandr
libxrandr-dev - X11 RandR extension library (development headers)
libxrandr2 - X11 RandR extension library
libxrandr2-dbg - X11 RandR extension library (debug package)
```

搜索后需要安装 `apt-get install libxrandr*` ，错误输出减少：

```bash
make[3]: Entering directory '/home/spice-h264/gstreamer-vaapi/tests'
  CCLD     simple-decoder
./.libs/libutils.a(libgstvaapi_x11_la-gstvaapidisplay_x11.o): In function `gst_vaapi_display_x11_get_size_mm':
/home/spice-h264/gstreamer-vaapi/gst-libs/gst/vaapi/gstvaapidisplay_x11.c:274: undefined reference to `XRRRootToScreen'
/home/spice-h264/gstreamer-vaapi/gst-libs/gst/vaapi/gstvaapidisplay_x11.c:276: undefined reference to `XRRGetScreenInfo'
/home/spice-h264/gstreamer-vaapi/gst-libs/gst/vaapi/gstvaapidisplay_x11.c:280: undefined reference to `XRRConfigCurrentConfiguration'
/home/spice-h264/gstreamer-vaapi/gst-libs/gst/vaapi/gstvaapidisplay_x11.c:292: undefined reference to `XRRFreeScreenConfigInfo'
/home/spice-h264/gstreamer-vaapi/gst-libs/gst/vaapi/gstvaapidisplay_x11.c:284: undefined reference to `XRRSizes'
./.libs/libutils.a(libgstvaapi_x11_la-gstvaapidisplay_x11.o): In function `check_extensions':
/home/spice-h264/gstreamer-vaapi/gst-libs/gst/vaapi/gstvaapidisplay_x11.c:118: undefined reference to `XRRQueryExtension'
collect2: error: ld returned 1 exit status
Makefile:789: recipe for target 'simple-decoder' failed
make[3]: *** [simple-decoder] Error 1
make[3]: Leaving directory '/home/spice-h264/gstreamer-vaapi/tests'
Makefile:1158: recipe for target 'all-recursive' failed
make[2]: *** [all-recursive] Error 1
make[2]: Leaving directory '/home/spice-h264/gstreamer-vaapi/tests'
Makefile:542: recipe for target 'all-recursive' failed
make[1]: *** [all-recursive] Error 1
make[1]: Leaving directory '/home/spice-h264/gstreamer-vaapi'
Makefile:474: recipe for target 'all' failed
make: *** [all] Error 2

```

### Left

- gstreamer-vaapi安装尚未成功
- doc还没过完

## 3-26

### To-Dos

- gstreamer-vaapi安装
- doc尽可能过完

### Logs

- gstreamer-vaapi

诡异的是，make通过了

- spice-protocol

同样很快的，编译通过了

- Setup Spice Server
- Build SPICE

打上patch，`./autogen.sh`有错，安装相应软件包

```bash
checking for ORC... no
configure: error: Package requirements (orc-0.4) were not met:

No package 'orc-0.4' found

Consider adjusting the PKG_CONFIG_PATH environment variable if you
installed software in a non-standard prefix.

Alternatively, you may set the environment variables ORC_CFLAGS
and ORC_LIBS to avoid the need to call pkg-config.
See the pkg-config man page for more details.
```

根据搜索结果安装`apt-get install liborc-0.4-*`

再根据输出，安装`celt051`，这个包没有在debian软件包里，需要手动编译安装

```bash
 cd /home/spice-h264
 wget https://src.fedoraproject.org/repo/pkgs/celt051/celt-0.5.1.3.tar.gz/67e7b5e45db57a6f1f0a6962f5ecb190/celt-0.5.1.3.tar.gz
 tar xzf celt-0.5.1.3.tar.gz
 cd celt-0.5.1.3
 ./configure --prefix=/home/spice-h264/resource/
 make && make install
```

然后再安装`apt-get install openssl`

但是还有下面的错误，同安装前一样：

```bash
checking for SSL... no
configure: error: Package requirements (openssl) were not met:

No package 'openssl' found

Consider adjusting the PKG_CONFIG_PATH environment variable if you
installed software in a non-standard prefix.

Alternatively, you may set the environment variables SSL_CFLAGS
and SSL_LIBS to avoid the need to call pkg-config.
See the pkg-config man page for more details.
```

额，居然安装了这个就过了`sudo apt-get install libssl-dev`

然后现在又有问题：

```bash
checking for jpeg_destroy_decompress in -ljpeg... no
configure: error: libjpeg not found
```

根据`apt-cache search`的结果：

```bash
root@haoyuan-Broadwell:/home/spice-h264/spice# apt-cache search libjpeg
libjpeg-dev - Independent JPEG Group's JPEG runtime library (dependency package)
libjpeg-turbo8 - IJG JPEG compliant runtime library.
libjpeg-turbo8-dbg - Debugging symbols for the libjpeg-turbo library
libjpeg-turbo8-dev - Development files for the IJG JPEG library
libjpeg8 - Independent JPEG Group's JPEG runtime library (dependency package)
libjpeg8-dbg - Independent JPEG Group's JPEG runtime library (dependency package)
libjpeg8-dev - Independent JPEG Group's JPEG runtime library (dependency package)
gem-plugin-jpeg - Graphics Environment for Multimedia - JPEG support
imgsizer - Adds WIDTH and HEIGHT attributes to IMG tags in HTML files
jp2a - converts jpg images to ascii
libjpeg-progs - Programs for manipulating JPEG files
libjpeg-turbo-progs - Programs for manipulating JPEG files
libjpeg-turbo-test - Program for benchmarking and testing libjpeg-turbo
libjpeg62 - Independent JPEG Group's JPEG runtime library (version 6.2)
libjpeg62-dbg - Development files for the IJG JPEG library (version 6.2)
libjpeg62-dev - Development files for the IJG JPEG library (version 6.2)
libjpeg9 - Independent JPEG Group's JPEG runtime library
libjpeg9-dbg - Development files for the IJG JPEG library
libjpeg9-dev - Development files for the IJG JPEG library
```

安装第一个`sudo apt-get install libjpeg-dev`

autogen成功：

```bash
configure:

        Spice 0.14.0.2-8f4b-dirty
        ==============

        prefix:                   /usr/local
        C compiler:               gcc

        LZ4 support:              no
        Smartcard:                no
        GStreamer:                1.0
        SASL support:             no
        Manual:                   no

        Now type 'make' to build spice

configure: WARNING: The avenc_mjpeg GStreamer element(s) are missing. You should be able to find them in the gstreamer-libav 1.0 package.

configure: WARNING: The vp8enc vp9enc GStreamer element(s) are missing. You should be able to find them in the gst-plugins-good 1.0 package.

configure: WARNING: The x264enc GStreamer element(s) are missing. You should be able to find them in the gst-plugins-ugly 1.0 package.

configure: WARNING: The GStreamer video encoder can be built but may not work.
```

之后的make没有成功，发现是patch没打完全，搞定之后安装成功

- x11spice

`./autogen.sh`的时候有包的缺失，安装之：

```bash
apt-get install libxcb-damage0* libxcb-xtest0* libxcb-xkb* 
```

### Left

- 以上安装待续，进行到了`No package 'gtk+-3.0' found`的阶段

## 3-27

### To-Dos

- 继续`gtk+-3.0`及后续的配置
- 过完Doc

### Logs

- `gtk+-3.0` 目前可以看到这些包

```bash
gir1.2-gtk-3.0 - GTK+ graphical user interface library -- gir bindings
libgtk-3-0 - GTK+ graphical user interface library
libgtk-3-0-dbg - GTK+ libraries and debugging symbols
gir1.2-javascriptcoregtk-3.0 - JavaScript engine library from WebKitGTK+ - GObject introspection data
gir1.2-spice-client-gtk-3.0 - GTK3 widget for SPICE clients (GObject-Introspection)
libjavascriptcoregtk-3.0-0 - JavaScript engine library from WebKitGTK+
libjavascriptcoregtk-3.0-0-dbg - JavaScript engine library from WebKitGTK+ - debugging symbols
libjavascriptcoregtk-3.0-bin - JavaScript engine library from WebKitGTK+ - command-line interpreter
libjavascriptcoregtk-3.0-dev - JavaScript engine library from WebKitGTK+ - development files
libspice-client-gtk-3.0-4 - GTK3 widget for SPICE clients (runtime library)
libspice-client-gtk-3.0-dev - GTK3 widget for SPICE clients (development files)
libwebkit2gtk-3.0-25 - WebKit2 API layer for WebKitGTK+
libwebkit2gtk-3.0-25-dbg - WebKit2 API layer for WebKitGTK+ - debugging symbols
libwebkit2gtk-3.0-dev - WebKit2 API layer for WebKitGTK+ - development files
libwebkitgtk-3.0-0 - Web content engine library for GTK+
libwebkitgtk-3.0-0-dbg - Web content engine library for GTK+ - debugging symbols
libwebkitgtk-3.0-common - Web content engine library for GTK+ - data files
libwebkitgtk-3.0-dev - Web content engine library for GTK+ - development files
```

根据描述安装：

```bash
apt-get install gir1.2-gtk-3.0 libgtk-3-0 libgtk-3-0-dbg gir1.2-spice-client-gtk-3.0 libspice-client-gtk-3.0-4 libspice-client-gtk-3.0-dev
```

`autogen.sh`成功

make成功

- spice-gtk

打了patch之后在`autogen.sh`时出错：

```bash
checking for valac... valac
configure: WARNING: no proper vala compiler found
configure: WARNING: you will not be able to compile vala source files
checking for vapigen... no
configure: error: Cannot find the "vapigen" binary in your PATH
```

然后我们根据apt-cache search的结果安装：

```bash
apt-get install valac*
```

之后成功并有如下提示：

```bash

configure:

        Spice-Gtk 0.34-dirty
        ==============

        prefix:                   /usr/local
        c compiler:               gcc
        Target:                   Unix

        Gtk:                      3.0
        Coroutine:                ucontext
        PulseAudio:               no
        GStreamer Audio:          yes
        GStreamer Video:          yes
        SASL support:             no
        Smartcard support:        no
        USB redirection support:  no
        DBus:                     yes
        WebDAV support:           no
        LZ4 support:              no

        Now type 'make' to build spice-gtk


configure: WARNING: The jpegdec vp8dec vp9dec GStreamer element(s) are missing. You should be able to find them in the gst-plugins-good 1.0 package.

configure: WARNING: The avdec_h264 GStreamer element(s) are missing. You should be able to find them in the gstreamer-libav 1.0 package.

configure: WARNING: The GStreamer video decoder can be built but may not work.

```

`make`出现下面的错误

```bash
Traceback (most recent call last):
  File "../spice_codegen.py", line 7, in <module>
    from python_modules import spice_parser
  File "/home/spice-h264/spice-gtk/spice-common/python_modules/spice_parser.py", line 1, in <module>
    import six
ImportError: No module named six
Makefile:810: recipe for target 'generated_client_demarshallers.c' failed
make[4]: *** [generated_client_demarshallers.c] Error 1
make[4]: Leaving directory '/home/spice-h264/spice-gtk/spice-common/common'
Makefile:456: recipe for target 'all-recursive' failed
make[3]: *** [all-recursive] Error 1
make[3]: Leaving directory '/home/spice-h264/spice-gtk/spice-common'
Makefile:388: recipe for target 'all' failed
make[2]: *** [all] Error 2
make[2]: Leaving directory '/home/spice-h264/spice-gtk/spice-common'
Makefile:628: recipe for target 'all-recursive' failed
make[1]: *** [all-recursive] Error 1
make[1]: Leaving directory '/home/spice-h264/spice-gtk'
Makefile:533: recipe for target 'all' failed
make: *** [all] Error 2
```

装了`python-six`后还是后面的那几个问题，于是我一气之下根据[Debian -- 在 buster 中的 spice-gtk 源码包详细信息](https://packages.debian.org/source/buster/spice-gtk)的软件包依赖部分安装了如下的软件包：

```bash
apt-get install  debhelper  libpixman-1-dev  libssl-dev  libgtk-3-dev   libglib2.0-dev  libgudev-1.0-dev  libcairo2-dev  libpulse-dev   libusb-1.0-0-dev   valac   python-all   python-six   python-gtk2-dev  python-pyparsing  libsasl2-dev  libjpeg-dev  gobject-introspection  libgirepository1.0-dev gir1.2-gtk-2.0 libtext-csv-perl   libusbredirhost-dev  libacl1-dev  libpolkit-agent-1-dev libpolkit-gobject-1-dev  dpkg-dev  libdbus-glib-1-dev libopus-dev  libsoup2.4-dev  gtk-doc-tools liblz4-dev  libcacard-dev  libspice-protocol-dev   libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gettext libepoxy-dev
```

于是成功了 ：）

- Test server & client

成功，但是之前忘记打gstreamer-vaapi的patch了，不知道是否调用到了，我们就假设是调用到了吧 ：）逃

### Left

- 结束文档的审阅工作，准备进入自己的Xspice工作

## 3-28

### To-Dos

- 审阅Xmind的毕设思维导图
- 分析x11spice / vaapi /h.264 porting 到XSpice的可能
- 做些许尝试

### Logs

N/A

### Left

All

## 3-29/30

### To-Dos

- 根据[README](https://cgit.freedesktop.org/xorg/driver/xf86-video-qxl/tree/README.xspice)来徒手配置Xspice
- 如果当日配置完成，分析哪些部分需要进行改进

### Log

见Linux上的`Build Xspice from Scratch.md`

### Left

- 在本机本地已经尝试可行，准备向Docker中迁移

## 4-02/03

### To-Dos

- 尽可能弄完Docker中的XSpice
- 继续寻找Web资源
- MySQL

### Logs

见`Build Xspice inside Docker Container.md`

### Left

- Windows环境下安装Django有错
- Linux环境下（Windows子系统）无法进行MySQL连接
- 在xserver部分卡住了，有个之前在native机子上没安装过的`libepoxy*`可能带来了问题