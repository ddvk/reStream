# reStream

reMarkable screen sharing over SSH.

[![rm1](https://img.shields.io/badge/rM1-supported-green)](https://remarkable.com/store/remarkable)
[![rm2](https://img.shields.io/badge/rM2-unsupported-red)](https://remarkable.com/store/remarkable-2)

![A demo of reStream](extra/demo.gif)

## Installation

1. Clone this repository: `git clone https://github.com/rien/reStream`.
2. (Optional but recommended) [Install lz4 on your host and reMarkable](#sub-second-latency).
3. [Set up an SSH key and add it to the ssh-agent](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent), then add your key to the reMarkable with `ssh-copy-id root@10.11.99.1`.

## RM2
get the rm2fb_xo from remarkable2-framebuff
```
systemct stop xochitl
LD_PRELOAD=/homer/toot/rm2fb_xofb.1.0.0 xochitl
```

## Usage

1. Connect your reMarkable with the USB cable.
2. Make sure you can [open an SSH connection](https://remarkablewiki.com/tech/ssh).
3. Run `./reStream.sh`
4. A screen will pop-up on your local machine, with a live view of your reMarkable!

### Options

- `-h --help`: show usage information
- `-p --portrait`: shows the reMarkable screen in portrait mode (default: landscape mode, 90 degrees rotated tot the right)
- `-s --source`: the ssh destination of the reMarkable (default: `root@10.11.99.1`)
- `-o --output`: path of the output where the video should be recorded, as understood by `ffmpeg`; if this is `-`, the video is displayed in a new window and not recorded anywhere (default: `-`)
- `-f --format`: when recording to an output, this option is used to force the encoding format; if this is `-`, `ffmpeg`’s auto format detection based on the file extension is used (default: `-`).
- `-w --webcam`: record to a video4linux2 web cam device. By default the first found web cam is taken, this can be overwritten with `-o`. The video is scaled to 1280x720 to ensure compatibility with MS Teams, Skype for business and other programs which need this specific format.
- `-t --throughput`: use `pv` to measure how much data throughput you have (good to experiment with parameters to speed up the pipeline)

If you have problems, don't hesitate to [open an issue](https://github.com/rien/reStream/issues/new) or [send me an email](mailto:rien.maertens@posteo.be).

## Requirements

On your **host** machine:
- Any POSIX-shell (e.g. bash)
- ffmpeg (with ffplay)
- ssh
- Video4Linux loopback kernel module if you want to use `--webcam`

On your **reMarkable** nothing is needed, unless you want...

### Sub-second latency

To achieve sub-second latency, you'll need [lz4](https://github.com/lz4/lz4)
on your host and on your reMarkable.

You can install `lz4` on your host with your usual package manager. On Ubuntu,
`apt install liblz4-tool` will do the trick.

On your **reMarkable** you'll need a binary of `lz4` build for the arm platform,
you can do this yourself by [installing the reMarkable toolchain](https://remarkablewiki.com/devel/qt_creator#toolchain)
and compiling `lz4` from source with the toolchain enabled, or you can use the
statically linked binary I have already built and put in this repo.

Copy the `lz4` program to your reMarkable with
`scp lz4.arm.static root@10.11.99.1:/home/root/lz4`, make it executable with
`ssh root@10.11.99.1 'chmod +x /home/root/lz4'` and you're ready to go.

### Video4Linux Loopback

To set your remarkable as a webcam we need to be able to fake one. This is where the Video4Linux loopback kernel module comes into play. We need both the dkms and util packages. On Ubuntu you need to install:

```
apt install v4l2loopback-utils v4l2loopback-dkms
```

In some package managers `v4l2loopback-utils` is found in `v4l-utils`.

After installing the module you must enable it with 

```
modprobe v4l2loopback
```

To verify that this worked, execute: 

```
v4l2-ctl --list-devices
```

The result should contain a line with "platform:v4l2loopback".

## Troubleshooting

Steps you can try if the script isn't working:
- [Set up an SSH key](#installation)
- Update `ffmpeg` to version 4.

