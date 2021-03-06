# Build XSpice from Scratch

Since I have to make some modification on how spice server is to call gstreamer, I need to build XSpice from source from scratch so that I could 1) know every steps and their dependencies, 2) understand the project structure and get a sense of where to modify 3) test if the [README](https://cgit.freedesktop.org/xorg/driver/xf86-video-qxl/tree/README.xspice) document is correct

## Set up the environment

Create directory

```bash
mkdir -p /home/xspice-test
mkdir -p /home/xspice-test/src
```

The followings commend are available as a script as well, see `env.sh`

Do remember to re-execute the script by `source env.sh` when you logout or switch tty.

*If you run into any problem, consider re-execute first, and see if the problem still exist.*

```bash
#! /bin/bash 
export TEST=/home/xspice-test
export PKG_CONFIG_PATH=${TEST}/lib/pkgconfig:${TEST}/share/pkgconfig
export MAKEFLAGS="-j4"
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

## Build | Make | Install Each of them

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

The following ?

### Build xorgproto

{已过期，请勿参阅

```bash
cd /home/xspice-test/src/xextproto
./autogen.sh --prefix=$TEST

XXX abandon

./configure --prefix=$TEST
make && make install
```



根据提示

```bash
root@haoyuan-Broadwell:/home/xspice-test/src/xextproto# ./autogen.sh --prefix=$TEST
This module has been deprecated. Use xorgproto instead:
git clone git://anongit.freedesktop.org/git/xorg/proto/xorgproto
```

于是根据该提示已经更改了`source env.sh`，重新clone了新的项目

}

Build x11proto[已被替代]

{已过期，请勿参阅

```bash
cd /home/xspice-test/src/x11proto
./autogen.sh --prefix=$TEST
./configure --prefix=$TEST
make && make install
```

又被deprecated了，并且替换为和上面相同的包

}

REAL THING

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

Before the actual building of xserver, we need to build `libdrm` on our own because the package in package source of my Ubuntu distribution is too old to meet the latest requirement of xserver.

- Build libdrm

    apt-get install libdrm-dev libdrm-intel1 libdrm2 libdrm-common

如果还出现问题我们可以使用apt-get upgrade后接相应的软件包进行升级，以满足条件，或者可以通过调整所build的xserver版本，使其对libdrm的版本要求降低。

本人配置的时候通过重新编译了一个libdrm来进行操作

    cd /home/xspice-test/src
    git clone https://anongit.freedesktop.org/git/mesa/drm.git/
    cd /home/xspice-test/src/drm
    ./autogen.sh --prefix=$TEST
    ./configure --prefix=$TEST
    make && make install

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

## Fire up and test

Both script and configure file are workable method to start xspice. If you use a simple script to fire up xspice, try:

- Inside xspice directory: `./scripts/Xspice --port 5900 --disable-ticketing :2.0`
- In another tty or terminal: `DISPLAY=:2.0 firefox &`or anything you want to run
- On the remote side, try `spicy` or `virt-viewer`to connect via `spice://IP:5900`

If you would like to use configure file, which support more detail setup to run xspice for enhanced functionality and performance, follow the official readme:

- Create directory for configure file:

```bash
mkdir -p $TEST/etc/X11
cp xspice/examples/spiceqxl.xorg.conf.example $TEST/etc/X11/spiceqxl.xorg.conf
```

- Set up Xorg related environment

```
== 3.1 Run Xorg directly ==
Run server with:
export XSPICE_PORT=5900
$XORG -noreset -config spiceqxl.xorg.conf :3.0

== 3.2 Run using the Xspice script ==
Or equivalently:

xspice/scripts/Xspice --port 5900 --disable-ticketing --xorg $XORG :3.0 [--vdagent]
(and many more options available -- see scripts/Xspice)

Run X clients as usual by setting DISPLAY=:3.0, for example
# DISPLAY=:3.0 firefox &  or 
# DISPLAY=:3.0 twm &  DISPLAY=:3.0 xterm &

Run spice client:
sudo yum install virt-viewer
remote-viewer spice://localhost:5900
```