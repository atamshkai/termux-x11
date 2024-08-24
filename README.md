
# Termux:X11

[![Nightly build](https://github.com/termux/termux-x11/actions/workflows/debug_build.yml/badge.svg?branch=master)](https://github.com/termux/termux-x11/actions/workflows/debug_build.yml) [![Join the chat at https://gitter.im/termux/termux](https://badges.gitter.im/termux/termux.svg)](https://gitter.im/termux/termux) [![Join the Termux discord server](https://img.shields.io/discord/641256914684084234?label=&logo=discord&logoColor=ffffff&color=5865F2)](https://discord.gg/HXpF69X)

A [Termux](https://termux.com) X11 server add-on app.

## About
Termux:X11 is a fully fledged X server. It is built with Android NDK and optimized to be used with Termux.

## Submodules caveat
This repo uses submodules. Use 

```
git clone --recurse-submodules https://github.com/termux/termux-x11 
```
or
```
git clone https://github.com/termux/termux-x11
cd termux-x11
git submodule update --init --recursive
```

## How does it work?
Just like any other X server.

## Setup Instructions
For this one you must enable the `x11-repo` repository can be done by executing `pkg install x11-repo` command

For X applications to work, you must install Termux-x11 companion package. You can do that by downloading an artifact from [last successful build](https://github.com/termux/termux-x11/actions/workflows/debug_build.yml) and installing `*.apk` and `*.deb` (if you use termux with `pkg`) or `*.tar.xz` (if you use termux with `pacman`) files.
Or you can install nightly companion package from repositories with `pkg in x11-repo && pkg in termux-x11-nightly`

## Running Graphical Applications
You can start your desired graphical application by doing:
```
termux-x11 :0 &
env DISPLAY=:0 dbus-launch --exit-with-session xfce4-session
```
or
```
termux-x11 :0 -xstartup "dbus-launch --exit-with-session xfce4-session"
```
You may replace `xfce4-session` if you use other than Xfce

`dbus-launch` does not work for some users so you can start session with
```
termux-x11 :0 -xstartup "xfce4-session"
```
Also you can do 
```
export TERMUX_X11_XSTARTUP="xfce4-session"
termux-x11 :0
```
In this case you can same TERMUX_X11_XSTARTUP somewhere in `.bashrc` or other script and not type it every time you invoke termux-x11.  


If you're done using Termux:X11 just simply exit it through it's notification drawer by expanding the Termux:X11 notification then "Exit"
But you should pay attention that `termux-x11` command is still running and can not be killed this way.

For some reason some devices output only black screen with cursor instead of normal output so you should pass `-legacy-drawing` option.
```
~ $ termux-x11 :0 -legacy-drawing -xstartup "xfce4-session"
```

For some reason some devices show screen with swapped colours, in this case you should pass `-force-bgra` option.
```
~ $ termux-x11 :0 -force-bgra -xstartup "xfce4-session"
```
### Sound system

#### Check your shell
```
echo $SHELL
```
#### If your shell is zsh,add this two lines to your ~/.zshrc
```
echo "killall pulseaudio &>/dev/null" >>~/.zshrc

echo "pulseaudio --start --exit-idle-time=-1; pacmd load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1" >>~/.zshrc
```
#### If your shell is bash
```
echo "killall pulseaudio &>/dev/null" >>~/.bashrc

echo "pulseaudio --start --exit-idle-time=-1; pacmd load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1" >>~/.bashrc
```
Then login again

If you're done using Termux:X11 just simply exit it through it's notification drawer by expanding the Termux:X11 notification then "Exit"
But you should pay attention that `termux-x11` command is still running and can not be killed this way.

## Using with proot environment

If you plan to use the program with proot, keep in mind that you need to launch proot/proot-distro with the --shared-tmp option. 
If passing this option is not possible, set the TMPDIR environment variable to point to the directory that corresponds to /tmp in the target container.
If you are using proot-distro you should know that it is possible to start `termux-x11` command from inside proot container.

### Start Termux-x11 First,

```
termux-x11 :0 &
```

### Then,open another session and login to proot-distro.

#### Example

```
proot-distro login ubuntu --shared-tmp
```

#### Now you can start it.
```
export PULSE_SERVER=127.0.0.1;env DISPLAY=:0 dbus-launch --exit-with-session xfce4-session &>/dev/null
```

#### Stop it.
```
pkill -f com.termux.x11
```

#### If you want to use Termux-X11 with your Custom OS,

#### use this example start-distro.sh

```
#!/data/data/com.termux/files/usr/bin/bash
cd $(dirname $0)
## unset LD_PRELOAD in case termux-exec is installed
unset LD_PRELOAD
command="proot"
command+=" --link2symlink"
command+=" -0"
command+=" -r ubuntu-fs"
if [ -n "$(ls -A binds)" ]; then
    for f in binds/* ;do
      . $f
    done
fi
command+=" -b /dev"
command+=" -b /proc"
command+=" -b /data/data/com.termux/files/usr/tmp"
command+=" -b ubuntu-fs/root:/dev/shm"
## uncomment the following line to have access to the home directory of termux
#command+=" -b /data/data/com.termux/files/home:/root"
## uncomment the following line to mount /sdcard directly to /
command+=" -b $PREFIX/tmp/.X11-unix:/tmp/.X11-unix"
command+=" -b /sdcard"
command+=" -w /root"
command+=" /usr/bin/env -i"
command+=" HOME=/root"
command+=" PATH=/usr/local/sbin:/usr/local/bin:/bin:/usr/bin:/sbin:/usr/sbin:/usr/games:/usr/local/games"
command+=" TERM=$TERM"
command+=" LANG=C.UTF-8"
command+=" /bin/bash --login"
com="$@"
if [ -z "$1" ];then
    exec $command
else
    $command -c "$com"
fi
```
## Using with chroot environment
If you plan to use the program with chroot or unshare, you must to run it as root and set the TMPDIR environment variable to point to the directory that corresponds to /tmp in the target container. 
This directory must be accessible from the shell from which you launch termux-x11, i.e. it must be in the same SELinux context, same mount namespace, and so on.
Also you must set `XKB_CONFIG_ROOT` environment variable pointing to container's `/usr/share/X11/xkb` directory, otherwise you will have `xkbcomp`-related errors.
You can get loader for nightly build from an artifact of [last successful build](https://github.com/termux/termux-x11/actions/workflows/debug_build.yml)
```
setenforce 0
export TMPDIR=/path/to/chroot/container/tmp
export CLASSPATH=$(/system/bin/pm path com.termux.x11 | cut -d: -f2)
/system/bin/app_process / com.termux.x11.CmdEntryPoint :0
```

### Another way to use Termux:x11 with Chroot Environment.

### Install termux-sudo
```
pkg up -y;pkg i -y git tsu
```

```
git clone https://gitlab.com/st42/termux-sudo
```

```
cd termux-sudo
```

```
cat sudo > /data/data/com.termux/files/usr/bin/sudo
```

```
chmod 700 /data/data/com.termux/files/usr/bin/sudo
```

#### Create a folder from termux if doesn't have it.

```
mkdir /data/data/com.termux/files/usr/tmp/.X11-unix
```

#### Link to Linux Distro's tmp (Example with nethunter)

```
tsu
```
```
ln -s /data/data/com.termux/files/usr/tmp/.X11-unix /data/local/nhsystem/kali-arm64/tmp/.X11-unix
```
```
exit
```

### Create a script in termux's bin

#### Example with nethunter

```
nano ~/../usr/bin/kangeli
```

### Add this to kangeli
```
#!/bin/bash
if [ $# -eq 0 ]
then
        echo "Empty argument, use --help to see available arguments"
elif [ $1 = "--start" ]
then
	unset LD_PRELOAD
	export ROOTFSPATH=/data/local/nhsystem/kali-arm64
	sudo mount -o remount,dev,suid /data
	sudo mount proc -t proc $ROOTFSPATH/proc
	sudo mount sys -t sysfs $ROOTFSPATH/sys
	sudo mount --bind /dev $ROOTFSPATH/dev
	sudo mount --bind /dev/pts $ROOTFSPATH/dev/pts
	sudo mount --bind /sdcard $ROOTFSPATH/sdcard
        echo "Starting Termux-x11..."
        termux-x11 :0 &>/dev/null & pulseaudio -k &>/dev/null & pulseaudio --start --load="module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1" --exit-idle-time=-1 &>/dev/null & sleep 1
        sleep 3
        echo ""
        echo -e "Termux-x11 started"
	sudo mount --bind $PREFIX/tmp/.X11-unix $ROOTFSPATH/tmp/.X11-unix
	echo ""
	echo "Starting ........"
        echo ""
	sudo chroot $ROOTFSPATH /bin/su -
	sudo umount $ROOTFSPATH/proc
	sudo umount $ROOTFSPATH/sys
	sudo umount $ROOTFSPATH/dev/pts
	sudo umount $ROOTFSPATH/dev
	sudo umount $ROOTFSPATH/sdcard
	sudo umount $ROOTFSPATH/tmp/.X11-unix
#	pkill -f "app_process / com.termux.x11"
fi
```

#### Give Permission
```
chmod +x ~/../usr/bin/kangeli
```

#### Open Termux-x11 
```
termux-x11 :0 &
```

#### Start Nethunter
```
tsu
```

```
kangeli --start
```

```
export PULSE_SERVER=127.0.0.1;export DISPLAY=:0;dbus-launch --exit-with-session startxfce4 &>/dev/null
```

#### Stop Nethunter
```
pkill -f com.termux.x11
```

## WARNING

#### If you open rootfs with one more users(nethunter & termux),permission error will happen on /tmp/.X11-unix

#### Solve Cannot Open Display Error about Permissions

#### In nethunter terminal
```
rm -rf /tmp/.* /tmp/*
```

```
mkdir /tmp/.X11-unix
```

```
exit
```

#### Test also In termux's tmp.If it is red,delete it.
```
sudo rm -rf /data/data/com.termux/files/usr/tmp/.X11-unix
```

```
mkdir /data/data/com.termux/files/usr/tmp/.X11-unix
```

### Then Open An Oridinary Termux-X11 Session.

#### Example
```
termux-x11 :0 -xstartup 'dbus-launch --exit-with-session startxfce4' &>/dev/null
```

### Then,close termux-x11 and restart terminal again
```
pkill -f com.termux.x11
```

```
exit
```

### Force stopping X server (running in termux background, not an activity)

termux-x11's X server runs in process with name "app_process", not "termux-x11". But you can kill it by searching "com.termux.x11" in commandline.
So killing it will look like
```
pkill -f com.termux.x11
```

### Closing Android activity (running in foreground, not X server)

```
am broadcast -a com.termux.x11.ACTION_STOP -p com.termux.x11
```
### Logs
If you need to obtain logs from the `com.termux.x11` application,
set the `TERMUX_X11_DEBUG` environment variable to 1, like this:
`TERMUX_X11_DEBUG=1 termux-x11 :0`

The log obtained in this way can be quite long.
It's better to redirect the output of the command to a file right away.

### Notification
In Android 13 post notifications was restricted so you should explicitly let Termux:X11 show you notifications.
<details>
<summary>Video</summary>

[img_enable-notifications.webm](https://user-images.githubusercontent.com/9674930/227760411-11d440eb-90b8-451e-9024-d5a194d10b16.webm)

</details>

Preferences:
You can access preferences menu three ways:
<details>
<summary>By clicking "PREFERENCES" button on main screen when no client connected.</summary>

![image](./img/1.jpg)
</details>
<details>
<summary>By clicking "Preferences" button in notification, if available.</summary>

![image](./img/2.jpg)
</details>
<details>
<summary>By clicking "Preferences" application shortcut (long tap `Termux:X11` icon in launcher). </summary>

![image](./img/3.jpg)
</details>

## Touch gestures
### Touchpad emulation mode.
In touchpad emulation mode you can use the following gestures:
* Tap for click
* Double tap for double click
* Two-finger tap for right click
* Three-finger tap for middle click
* Two-finger vertical swipe for vertical scroll
* Two-finger horizontal swipe for horizontal scroll
* Three-finger swipe down to show-hide additional keys bar.
### Mouse emulation mode.
In touchpad emulation mode you can use the following gestures:
* Mouse is in click mode as long as you hold finger on a screen.
* Double tap for double click
* Two-finger tap for right click
* Three-finger tap for middle click
* Two-finger vertical swipe for vertical scroll
* Two-finger horizontal swipe for horizontal scroll
* Three-finger swipe down to show-hide additional keys bar.

## Font or scaling is too big!
Some apps may have issues with X server regarding DPI. please see https://wiki.archlinux.org/title/HiDPI on how to override application-specific DPI or scaling.

You can fix this in your window manager settings (in the case of xfce4 and lxqt via Applications Menu > Settings > Appearance). Look for the DPI value, if it is disabled enable it and adjust its value until the fonts are the appropriate size.
<details>
<summary> Screenshot </summary>

![image](./img/dpi-scale.png) 
</details>

Also you can start `termux-x11` with `-dpi` option.
```
~ $ termux-x11 :0 -xstartup "xfce4-session" -dpi 120
```

## Using with 3rd party apps
It is possible to use Termux:X11 with 3rd party apps.
Check how `shell-loader/src/main/java/com/termux/x11/Loader.java` works.

[Download](https://github.com/termux/termux-x11/releases
) 

# License
Released under the [GPLv3 license](https://www.gnu.org/licenses/gpl-3.0.html).
