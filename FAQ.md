## Android Apps ##
### How do I find the target's IP address? ###
  * On the target terminal, "`netcfg`" - if you don't see the IP address, try "`netcfg eth0 dhcp`"
  * If you can't bring up the Ethernet, double check that the ethaddr was set up u-boot (i.e. "`set ethaddr 00:50:c2:xx:xx:xx`")

### How do I connect via adb? ###
  * Assuming you have installed the Android SDK and <android-sdk-x.xx>/tools is on your PATH:
    * `$ ADBHOST=xxx.xxx.xxx.xxx adb kill-server` (note ADBHOST is the IP address of the target)
    * `$ ADBHOST=xxx.xxx.xxx.xxx adb shell`

### How do I install an app? ###
  * `$ ADBHOST=xxx.xxx.xxx.xxx adb -s device-5554 install <my/path/to>/CoolApp.apk`
  * You can see the list of all connected devices with "`adb devices`"