# Install Unofficial CyanogenMod to Galaxy Tab 4 7.0 SM-T230NU

This archives the files you need and gets through all the hurdles of the install.  This is geared towards linux users, but could probably be adapted for Windows/Apple users.

## Clone Repo and Extract

    # install unar
    sudo apt install unar

    # clone
    git clone https://github.com/pjobson/galaxy_tab_4_7.0_CM.git
    cd galaxy_tab_4_7.0_CM

    # extract
    tar xzvf odin4.tar.gz
    chmod +x odin4
    gunzip Adaway.apk.gz
    gunzip Nova-Launcher.apk.gz
    gunzip Ginger_Keyboard.apk.gz
    gunzip twrp-3.7.0_9-0-degas.tar.gz # don't untar it
    unar cm/cm.part01.rar
    unar oga/oga.part01.rar
    # remove these to save space
    rm -rf oga/
    rm -rf cm/

## TWRP

In your host system:

To the top of your `/etc/udev/rules.d/51-android.rules` add:

    SUBSYSTEM=="usb", ATTR{idVendor}=="04e8", MODE="0666", GROUP="plugdev"

The restart udevadm.

    sudo udevadm trigger

Make sure your user is in the plugdev group, you'll need to reboot or login/logout after this.

    sudo usermod -a -G plugdev $USER

Boot into Download mode (Volume Down, Home Btn, Power), install TWRP.

    ./odin4 -b twrp-3.7.0_9-0-degas.tar

This may automatically boot to recovery, if not then get back with
`adb reboot recovery` or Volume Up, Home Btn, Power.

## Manually Format

For some reason TWRP doesn't format properly.

Check device connection

    adb devices

Unmount all partitions

    adb shell umount /sdcard
    adb shell umount /data
    adb shell umount /cache

Format partitions

    # CACHE
    adb shell make_ext4fs /dev/block/mmcblk0p14
    # SYSTEM
    adb shell make_ext4fs /dev/block/mmcblk0p15
    # HIDDEN/preload
    adb shell make_ext4fs /dev/block/mmcblk0p13
    # USER/data
    adb shell make_ext4fs /dev/block/mmcblk0p16

Remount and verify

    adb shell mount -t ext4 /dev/block/mmcblk0p14 /cache
    adb shell mount -t ext4 /dev/block/mmcblk0p15 /system
    adb shell mount -t ext4 /dev/block/mmcblk0p13 /preload
    adb shell mount -t ext4 /dev/block/mmcblk0p16 /data

    adb shell df -h /cache /system /preload /data

## Install CyanogenMod 11 Unofficial

From your host machine.

    adb push cm-11-20211124-UNOFFICIAL-degaswifi.zip /sdcard
    adb push OpenGapps-degas.zip /sdcard
    adb push SuperSU-v2.82-201705271822.zip /sdcard

In TWRP, select INSTALL, then add each one of those zip files. Only add
SuperSU if you want your device rooted, really if you're using this old
device I'd be surprised if you didn't.

If this doesen't automatically reboot to system, do it manually.
Sometimes the screen will just go black, you can do `adb reboot`.

At this point when it comes back up, the AOSP keyboard will not work, so
just go through the basic setup.  We'll get to fixing it shortly.

## Re-Enable Developer Mode

Settings -> General -> About Device -> Tap Build Number 7 times

## Re-Enable USB Debugging

Settings -> General -> Developer Options

* Check USB Debugging

There's a bug here where it won't ask you for your key... for reasons I
guess.

Reboot back to TWRP Vol Up, Pwr, Home.

Manually add your key.

    adb push ~/.android/adbkey.pub /sdcard
    adb shell mv /sdcard/adbkey.pub /data/misc/adb/adb_keys
    adb reboot
    # wait for android to start
    adb kill-server
    adb devices
    adb shell

## Apps

The AOSP keyboard is broken, no clue why.

    adb install Ginger_Keybaord-9.7.3.apk

Settings -> Controls -> Language and Input

* Set keyboard to Ginger Keyboard
* Click Ginger Keyboard Settings
* Click Languages
* Make sure this is set to Ginger Keyboard, mine was AOSP.
* Close settings out.

Disable AOSP keyboard (it'll crash constantly if you don't).

    adb shell
    pm block com.android.inputmethod.latin
    pm block com.android.inputdevices
    exit

Note: You can block other apps if you're not going to use them.

    pm block com.android.browser
    pm block com.android.email
    pm block com.android.calendar

I included Adaway and NovaLauncher, this is the Nova from before they got bought out and included a bunch of spyware.

    adb install Nova-Launcher.apk
    adb install Adaway.apk

If you don't want google, you can install apps from ApkMirror.
F-Droid doesn't work for this device because it is too old I guess.

You can only install apps for Android <= 4.4.4

* Firefox - https://www.apkmirror.com/apk/mozilla/firefox/variant-%7B%22arches_slug%22%3A%5B%22armeabi%22%5D%2C%22dpis_slug%22%3A%5B%22nodpi%22%5D%2C%22minapi_slug%22%3A%22minapi-8%22%7D/
* Signal - https://www.apkmirror.com/apk/signal-foundation/signal-private-messenger/
* VLC - https://www.apkmirror.com/apk/videolabs/vlc/

## Credits

* Unofficial CM11 - https://xdaforums.com/t/sm-t230-sm-t230nu-sm-t231-unofficial-cm-11-cm-14-1-based-android-4-4-4-7-1-2.3648887/
* Odin4 - https://github.com/Llucs/odin4
* SuperSU - https://supersuroot.org/
* OpenGapps - https://archive.org/details/open_gapps-arm-4.4-nano-20220215
