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



### Left

