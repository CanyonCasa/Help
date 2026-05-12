# Raspberry Pi Image Creation
How to for creating an RPi img file on a running RPi.

### Keywords
Raspberry Pi, RPi, img, remote, local, help

## Remote
From a remote RPi to a local Linux machine (e.g. a second RPi with adequate disk capacity).

1. Install netcat ON BOTH machines if necessay ...

``` bash
sudo apt install netcat-traditional
```

2. On the local Linux machine run ...

``` bash
sudo nc -l -p 7000 | dd of=<img_filename>
```
3. On remote RPi run ...

``` bash
sudo dd if=/dev/mmcblk0 | nc [local_machine_IP] 7000
```

    Note: you may need to Ctrl-C when complete to terminate the process.

The netcat program works like the cat command over a network socket. The first command (step 2) sets up a listening socket on the local machine and pipes the received data to a dd command which writes it to a specified output file. The second command launches dd on the remote RPi and copies the SD card (i.e. /dev/mmcblk0) to standard output, which pipes to netcat which in turn writes to the socket on the local machine.

After the transfer completes optionally shrink the image (and coompress it) to ensure it fits on a similar sized SD card. 

1. Install pishrink shell if necessary ...

``` bash
wget https://raw.githubusercontent.com/Drewsif/PiShrink/master/pishrink.sh
chmod +x pishrink.sh
sudo mv pishrink.sh /usr/local/bin
```

2.  Shrink and compress image to .img.gz file ...

``` bash
sudo pishrink.sh -Zv <img_filenaem> <shrunk_img_filename>
```
    Note: this can shrink a 16GB img to less than 2GB!

## Local
On the same RPi but to a separate drive (i.e. a USB thumb drive).

If enough room exists on a second drive to hold the image the dd command can work directly ...

``` bash
sudo dd if=/dev/mmcblk0 dd of=/dev/<drive>/<img_filename>
```
