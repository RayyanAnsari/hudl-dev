## The Tesco Hudl (HT7S3)
The Tesco Hudl is a tablet launched by British retailer Tesco in 2013. The device features a seven-inch screen (1440x900), a 1.5 GHz quad-core processor (the RK3188 with Mali 400) and 16 GB of internal flash memory. The Hudl runs the Android Jelly Bean operating system (Android 4.2.2).
|                     |                                                                                                                              |
|---------------------|------------------------------------------------------------------------------------------------------------------------------|
| Developer           |	Tesco                                                                                                                        |
| Manufacturer        |	Wistron Corporation                                                                                                          |
| Release date	      | September 30, 2013                                                                                                           |
| Introductory price	| £119                                                                                                                         |
| Discontinued	      | October 22, 2015                                                                                                             |
| Operating system	  | Android Jelly Bean                                                                                                           |
| System on a chip	  | Quad-core Rockchip RK3188 w/ Mali 400 Graphics                                                                               |
| Memory              |	1 GB RAM                                                                                                                     |
| Storage	            | 16 GB flash memory                                                                                                           |
| Display	            | IPS 7-inch (18 cm) LCD display<br/>1440 × 900 px (242 ppi; 95.3 ppcm) (WSXGA)<br/>16:10 aspect ratio<br/>(1080p HDMI output) |
| Camera	            | 720p HD video; 3 MP rear (autofocus)<br/>2 MP front (fixed-focus)                                                            | 
| Connectivity	      | Wi-Fi (802.11 a/b/g/n), Bluetooth 4.0, Micro-USB, Micro-HDMI                                                                 |

### Building the Tesco Hudl's Kernel
Tesco published the Hudl's kernel source on [this page](https://web.archive.org/web/20160322105950/https://www.tescotechsupport.com/downloads/). However, that website seems to be down now. Luckily for us, the kernel source has been uploaded to other places and archived on the Internet Archive. I also have a [mirror of this up on GitHub](https://github.com/WaluigiWare64/hudl-kernel/tree/stock). However, during compilation this errors out due to deprecated useage in the `timeconst.pl` Perl file. [I have created a branch](https://github.com/WaluigiWare64/hudl-kernel/tree/patched) called `patched` which fixes this issue.
```
git clone https://github.com/WaluigiWare64/hudl-kernel
git clone https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/arm/arm-eabi-4.8
export ARCH=arm CROSS_COMPILE=$(pwd)/arm-eabi-4.8/bin/arm-eabi-
cd tesco-hudl
git checkout patched
make rk3188_qc880_defconfig
make -j$(nproc --all)
make zkernel.img
mkdir -p output
cp kernel.img arch/arm/boot/zImage output
```

#### Flashing it to the Hudl
The Hudl has a Rockchip RK3188 SoC, and it uses the RK Batch Tool program. However, an open source alternative has been developed, called rkflashtool. There are two ways we can flash the kernel - embedding it in the boot image, or flashing it into the kernel partition. The Hudl has an 8 MiB kernel partition, however it seems to be unused. The stock configuration is:
- Boot partition with ramdisk and embedded kernel
- Recovery partition with recovery and embedded kernel
- Empty kernel partition

It is preferred to use the kernel partition, as new kernels can be flashed independently without the need to unpack and repack the boot image. In the section above, a folder named "output" was created, which contains a "kernel.img" and "zImage".
First, we need to unpack the boot image to remove the embedded kernel. The Hudl can be placed into the bootloader mode by using `adb reboot bootloader`. You can also hold the volume up button and press the reset button on the back with a pin.
To read the boot image we need to use rkflashtool. You can install this through your distro's repositories, or build this yourself.
```
sudo rkflashtool r boot > stock_boot.img
```
Keep the stock_boot.img safe as you will need it if you want to restore your device to stock.
```
curl -o split_bootimg.pl https://www.enck.org/tools/split_bootimg_pl.txt
perl split_bootimg.pl stock_boot.img
rkcrc -k boot.img-ramdisk.gz repacked_boot.img
```
Now we have a `repacked_boot.img` and a `kernel.img`. This can be flashed to the Hudl like so:
```
sudo rkflashtool w boot < repacked_boot.img
sudo rkflashtool w kernel < kernel.img
```
To reboot the Hudl use `sudo rkflashtool b`.

And we are done! Check the Hudl's About section in Settings and you should see the kernel info along with the compiler that was used.
