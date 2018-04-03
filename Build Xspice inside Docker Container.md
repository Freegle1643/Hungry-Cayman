# Build Xspice inside Docker Container

This is a based on `Build XSpice from Scratch.md`, we are to test if Xspice could be successfully built inside Docker, so that future modification will work. 

## Set up an image

First of all, we set up an Ubuntu image based on `ubuntu:16.04`.  Here we don't need to have many arguments to run the container, simply fire it up and install necessary software inside it.

`docker run -it ubuntu:16.04`

```bash
apt-get update
apt-get install sudo wget curl net-tools git vim \
				dh-autoreconf pkg-config \
				glib* libpixman-1-* bison \
				gtk-doc* gobject-introspection libglib2.0-dev libxrandr* libjpeg-dev\ 
				liborc-0.4-* libssl-dev libxcb-damage0* libxcb-xtest0* libxcb-xkb* \
				python-all   python-six   python-gtk2-dev  python-pyparsing \
				texlive-font-utils xfonts-utils
```

## Git Clone Every Source Code

The followings commend are available as a script as well, see `gitclone.sh`

```bash
#! /bin/bash 
git clone https://anongit.freedesktop.org/git/xorg/proto/xorgproto.git/  
git clone https://anongit.freedesktop.org/git/xorg/proto/fontsproto.git/ 
git clone https://anongit.freedesktop.org/git/xorg/app/xkbcomp.git/ 
git clone https://anongit.freedesktop.org/git/xorg/xserver.git/ 
git clone https://anongit.freedesktop.org/git/xorg/lib/libxtrans.git/ 
git clone https://anongit.freedesktop.org/git/xorg/lib/libxkbfile.git/ 
git clone https://anongit.freedesktop.org/git/xkeyboard-config.git/ 
git clone https://anongit.freedesktop.org/git/spice/spice-protocol.git/ 
git clone https://anongit.freedesktop.org/git/spice/spice.git/ 
git clone git://anongit.freedesktop.org/xorg/driver/xf86-video-qxl xspice
```

### Build spice

Following two sections

### Build spice-protocol

```bash
cd /home/xspice-test/src/spice-protocol
git checkout v0.12.13
./autogen.sh --prefix=$TEST
./configure --prefix=$TEST
make && make install
```

### Build spice

Try to install the following if you find any problem [**Already included above**]

```bash
apt-get install glib* libpixman-1-* \
		gtk-doc* gobject-introspection libglib2.0-dev libxrandr* \
		liborc-0.4-* libssl-dev libxcb-damage0* libxcb-xtest0* libxcb-xkb* libjpeg-dev \
		python-all   python-six   python-gtk2-dev  python-pyparsing
```



The reason to `git checkout 0.14` is that the newest branch of code require `spice-protocol >= 0.12.14`, but what we build above is 0.12.13. 

```bash
cd /home/xspice-test/src/spice
git checkout 0.14
./autogen.sh --prefix=$TEST
./configure --prefix=$TEST
make && make install
```

During the `./autogen.sh` process, some requirement were not met, which could be solved through following commend. Here we install celt051:

```bash
cd /home/xspice-test/src
wget http://downloads.us.xiph.org/releases/celt/celt-0.5.1.3.tar.gz
tar xzf celt-0.5.1.3.tar.gz
cd celt-0.5.1.3
./configure --prefix=$TEST
make && make install
```

After successfully installed celt051, go back to `./autogen.sh --prefix=$TEST` of spice, and you should be able to finish the process.

### Build xserver

The following X

### Build xorgproto

`apt-get install xutils-dev `

```bash
cd /home/xspice-test/src/xorgproto
./autogen.sh --prefix=$TEST
./configure --prefix=$TEST
make && make install
```

### Build libxtrans

```bash
cd /home/xspice-test/src/libxtrans
./autogen.sh --prefix=$TEST
./configure --prefix=$TEST
make && make install
```

### Build libxkbfile

```bash
cd /home/xspice-test/src/libxkbfile
./autogen.sh --prefix=$TEST
./configure --prefix=$TEST
make && make install
```

### Build xkbcomp

`apt-get install bison`

```bash
cd /home/xspice-test/src/xkbcomp
./autogen.sh --prefix=$TEST
./configure --prefix=$TEST
make && make install
```

### Build xkeyboard-config

```bash
cd /home/xspice-test/src/xkeyboard-config
./autogen.sh --prefix=$TEST
./configure --prefix=$TEST
make && make install
```

### Build xserver

`apt-get install texlive-font-utils xfonts-utils ` 

Before the actual building of xserver, we need to build `libdrm` on our own because the package in package source of my Ubuntu distribution is too old to meet the latest requirement of xserver.

- Build libdrm

```
apt-get install libdrm-dev libdrm-intel1 libdrm2 libdrm-common
```

如果还出现问题我们可以使用apt-get upgrade后接相应的软件包进行升级，以满足条件，或者可以通过调整所build的xserver版本，使其对libdrm的版本要求降低。

本人配置的时候通过重新编译了一个libdrm来进行操作

`apt-get install libpciaccess* libgl1-mesa-dev libepoxy* ` 

```
cd /home/xspice-test/src
git clone https://anongit.freedesktop.org/git/mesa/drm.git/
cd /home/xspice-test/src/drm
./autogen.sh --prefix=$TEST
./configure --prefix=$TEST
make && make install
```

- Now we build xserver

According to [README line 115](https://cgit.freedesktop.org/xorg/driver/xf86-video-qxl/tree/README.xspice#n115), there are a few more steps required before build xserver

To do so, create a temporary script `temp.sh` in `/home/xspice-test/src/xserver`

```sh
#! /bin/bash 
ADDSTR="#ifndef DBUS_TYPE_UNIX_FD\n"
ADDSTR+="#define DBUS_TYPE_UNIX_FD  ((int) 'h')\n"
ADDSTR+="#endif"
sed -i "/define DBUS_TIMEOUT/ a$ADDSTR" \
   ./hw/xfree86/os-support/linux/systemd-logind.c
```

Use `source temp.sh` to run the script. This will define `DBUS_TYPE_UNIX_FD` environment variable.

Then we need to set `ACLOCAL` for libxtrans

`export ACLOCAL="aclocal -I $TEST/share/aclocal"`

Now we can use the following to build xserver

```bash
cd /home/xspice-test/src/xserver
./autogen.sh --prefix=$TEST
./configure --prefix=$TEST
make && make install
```

### Build xspice

```bash
cd /home/xspice-test/src/xspice
./autogen.sh --prefix=$TEST --enable-xspice
./configure --prefix=$TEST --enable-xspice
make && make install
```

If you have error output like below

```bash
checking whether to build static libraries... no
./configure: line 18772: syntax error near unexpected token `RANDR,'
./configure: line 18772: `XORG_DRIVER_CHECK_EXT(RANDR, randrproto)'
```

Try `apt-get install xorg-dev` to solve the problem