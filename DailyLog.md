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

根据网上查询的资料，需要使用如下命令先安装一些应用程序

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

根据`apt-cache search`的搜寻， 又安装了`gtk-doc*`，再次运行出现下列报错：

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



### Left

