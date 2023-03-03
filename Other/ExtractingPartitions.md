# Extract the boot.img file or other partitions

## Files/Tools Needed 📃

- TWRP image: [surfaceduo1-twrp.img](https://github.com/WOA-Project/SurfaceDuo-Guides/raw/main/InstallWindows/Files/surfaceduo1-twrp.img)
- [Platform Tools from Google (ADB and Fastboot)](https://developer.android.com/studio/releases/platform-tools)
- A Windows PC

## Warnings ⚠️

- Read the entire guide before starting. Make sure you understand all of what you're going to do!
- If you see a warning and/or error during the process, it is not normal. Contact us on telegram if you see anything odd, but do not continue or proceed on your own, you will break things further.
- Don't rerun the commands if you interrupt the process. You may break things. 
- Do not run all commands at once.
- Do not commit *any* typo with *any* commands. 
- Be familiar with command line interfaces. 
- When using TWRP, it is normal and expected for the phone to be detected as a Xiaomi phone and for touch to not work.

## Getting your current boot slot (A/B)

Boot to Android™, on your PC open a command prompt window and type:

```
adb reboot fastboot
```

Your Duo will be rebooted to fastbootd (not fastboot). Once you're there:

```
fastboot getvar all
```

A lot of text lines will be printed. Look for a line that says `(bootloader) current-slot:`. It can be a or b. In my case:

<img width="204" alt="image" src="https://user-images.githubusercontent.com/29689637/222553048-6d473cbe-820a-452d-a77c-5b42cbd2682b.png">

Take note of your slot, we'll need that later. For the rest of this guide we'll assume it is `b`.

Now let's get back into Android™:

```
fastboot reboot
```

## Extracting the boot or other partitions

For the rest of this guide we'll pretend we need to extract the `boot` partition and that our current slot is `b`.

Once we're back in Android™:

```
adb reboot bootloader
```

When the bootloader menu shows up, boot TWRP:

```
fastboot boot twrp.img
```

Wait until TWRP finishes loading. Friendly reminder that touch still doesn't work in TWRP.

Now we'll need to find the location of our boot partition, as it is different for each device.

On the command prompt:

```
adb shell
```

This will open a shell on our device and show its output on our PC. 

```
ls /dev/block/platform/soc/
```

If you're lucky, you'll get only one line of output:

<img width="280" alt="image" src="https://user-images.githubusercontent.com/29689637/222556387-7595aa9c-e452-4534-afb2-acd2515e9496.png">

Take note of all the lines that get output from this command. In my case, it's only one and it is `1d84000.ufshc`.

Back into the shell:

```
ls -al /dev/block/platform/soc/[the last line we took note of]/by-name/
```

*In my case the command will be: `ls -al /dev/block/platform/soc/1d84000.ufshc/by-name/`*

This is going to output a lot of lines, with each partition name having its mount point. Look for the line that says `boot_[your slot]`:

<img width="506" alt="image" src="https://user-images.githubusercontent.com/29689637/222557262-7cbe0114-a218-4d60-9da3-2e04a9733bd6.png">

Now let's take note of its mount point, in my case: `/dev/block/sde26`.

Let's make an image of the partition:

```
dd if=/dev/block/[your mount point] of=/tmp/boot.img
```
*In my case, the command will be: `dd if=/dev/block/sde26 of=/tmp/boot.img`.*

Now let's exit the shell and extract the boot.img from the device:

```
exit

adb pull /tmp/boot.img
```

We're done! 🥳🎈

<img width="531" alt="image" src="https://user-images.githubusercontent.com/29689637/222561203-7f2dc375-e500-4201-a538-8cd0ba6d7559.png">

## Extracting and copying modem files (Carrier unlocked US AT&T)

If you are using a US AT&T device that was carrier unlocked, you will see in the [Status update](https://github.com/WOA-Project/SurfaceDuo-Guides/blob/5756da9ba9d57fff4b776202b858cc07f7e3887e/Status.md) that modem files are required from the Android image. Note that the device MUST ALREADY BE UNLOCKED IN ANDROID for this to work. Following this guide, use: 

```
ls -al /dev/block/platform/soc/[the last line we took note of]/by-name/
```

To generate the list of partitions and identify partitions modemst1 and modemst2. Note these partition numbers and use the appropriate commands:

```
dd if=/dev/block/sdf2 of=/tmp/bootmodem_fs1
dd if=/dev/block/sdf3 of=/tmp/bootmodem_fs2
```

Replacing sdf2/3 with the appropriate partitions. For my Surface Duo 1, the two image files are 2048 kb and 104 kb. The 104 kb file appears to be for the ESIM that is deactivated by AT&T in the US AT&T SD1. 

Next, use:

```
adb pull /tmp/bootmodem_fs1
adb pull /tmp/bootmodem_fs2
```

As above. Now, using the commands in the installation guide, we need to get into mass storage mode.

```
adb reboot bootloader
adb push surfaceduo1-msc.tar /sdcard/
adb shell "tar -xf /sdcard/surfaceduo1-msc.tar -C /sdcard --no-same-owner"
adb shell "chmod +x /sdcard/msc.sh"
adb shell "/sdcard/msc.sh"
```

Using the appropriate .tar file that you have from the installation process.

Now find the directory \Windows\System32\DriverStore\FileRepository\qcremotefs8150_<random data here> - making sure to look in your mounted Surface Duo and not your desktop.
  
Once you find this folder, select the folder (within FileRepository), right click from your desktop computer, and select security, and give full permissions to "EVERYONE."
  
Now you can rename the bootmodem_fs1 and _fs2 files in this folder (note they have no extension - you can add .old), and then paste the bootmodem_fs1 and _fs2 files you pulled from Android, above, into this folder. 
  
You can go back to the folder and restore the permissions (deselect modify/write permissions). 
  
If you boot into Windows and you get a BSOD/GSOD, then try swapping which of the two files are labeled fs1 and fs2. For me, the larger (2048kb) file needed to be fs1. When you come back up in Windows, you should have access to the cellular modem. You may have to adjust APN settings in Windows. The error message saying that the device is Carrier Locked should no longer appear. 
