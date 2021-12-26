# chipbox-dropbear

dropbear in chipbox

## History

Back in 2011, when I first implemented dropbear in Chipbox ( dropbear 0.52 ), I was using vsftpd to transfer files, and wasn't using SCP...

After 10 years, when I started working back on Chipbox, I decided to have a look and then found that scp wasn't working at all !!

So I decided to give a try.

## Repo structure

In this repo, you will find latest stable dropbear ( 2020-81 ) tarball, decompressed & compiled versions, as well as old dropbear0.52 used in chipbox.

## How did I compiled dropbear-2020.81 to chipbox?

1. Export PATH for toolchain

```
export PATH=/media/DATA/MERIH-YEDEK/toolchains/crosstool/gcc-3.4.6-glibc-2.3.6/arm-linux/bin/:$PATH
```

2. cd to new dropbear and compile as follows:

```
CC=arm-linux-gcc ./configure --host=arm-linux
```

Note: If I add ```--enable-static``` to configure line above, it creates a statically compiled dropbearmulti binary, and it works as well. But I decided to give a try to shared lib mode, and the size reduced to 226Kbyte.

```
make PROGRAMS="dropbear dropbearkey scp" MULTI=1 strip
```
This created a statically compiled & stripped binary:

```
$  ls -lash dropbearmulti 
228K -rwxrwxr-x 1 stulluk stulluk 226K Ara 26 14:06 dropbearmulti
$  file dropbearmulti 
dropbearmulti: ELF 32-bit LSB executable, ARM, version 1 (ARM), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.4.18, stripped
$  md5sum dropbearmulti 
02307e4f61bf08fa163430ddbc0902e1  dropbearmulti

```

3. upload dropbearmulti to chipbox /usr/plugin partition ( because I had free space there )
4. create symlinks
```
ln -s /usr/plugin/dropbearmulti /usr/plugin/dropbear
ln -s /usr/plugin/dropbearmulti /usr/plugin/dropbearkey
ln -s /usr/plugin/dropbearmulti /usr/bin/scp
```

5. system init:

There is already an init script under /etc/init.d:
```
# cat /etc/init.d/S90dropbear 
#!/bin/sh
#
# Start the dropbear....
#

start() {
        echo "Starting dropbear..."
        /usr/plugin/dropbear -b /usr/plugin/dr_banner.txt
}
stop() {
        echo -n "Stopping dropbear..."
        killall dropbear
}
restart() {
        stop
        start
}

case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  restart|reload)
        restart
        ;;
  *)
        echo $"Usage: $0 {start|stop|restart}"
        exit 1
esac

exit $?

# 
```

Then I decided to remove the banner, and run a small test:

```
stulluk ~ $  scp IMG_20211206_153915.jpg cbx:/tmp/
Warning: Permanently added '192.168.1.232' (RSA) to the list of known hosts.
IMG_20211206_153915.jpg                                               100% 4256KB 742.6KB/s   00:05    
stulluk ~ $
```

Sadly, too slow... :(

## Further considerations

When I run df command, it doesn't immediately show free space, I need to wait a while. I feel the RW speed of mtdblock or NOR Flash is too slow, so it doesn't reflect changes quickly.


