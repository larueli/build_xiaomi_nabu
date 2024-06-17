# TWRP Build

## Add twrp sources

Based on https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp

```
cd
mkdir twrp
cd twrp
repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-12.1 --depth=1
```

## Add nabu sources

Based on

* https://github.com/SIDDK24/device_xiaomi_nabu device tree,
* https://github.com/LesGaR/android_kernel_xiaomi_nabu (13.0 branch) kernel tree

Beware : most known https://github.com/Kfkcome/android_kernel_xiaomi_nabu not working, even on 13.0 branch : we have a blue screen, only display part of notification bar. The fork by LesGaR is working (on 13 branch), it has several commits about fixing recovery, fxing refresh rate in twrp (9f2eaba, da708a6) so I think that's why only this one works.

`mkdir .repo/local_manifests` then `nano .repo/local_manifests/roomservice.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <!--Device tree-->
  <project name="SIDDK24/device_xiaomi_nabu" path="device/xiaomi/nabu" remote="github" revision="android-12.1"/>
  <!--Kernel-->
  <project name="LesGaR/android_kernel_xiaomi_nabu" path="kernel/xiaomi/nabu" remote="github" revision="13.0"/>
</manifest>
```

## Fetch source

`repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags`

## Build

```
export ALLOW_MISSING_DEPENDENCIES=true
. build/envsetup.sh
lunch twrp_nabu-eng
export USE_CCACHE=1 && export CCACHE_EXEC=/usr/bin/ccache && ccache -M 60G && export CCACHE_DIR=/mnt/ccache
mka bootimage
```

Get the boot.img using scp, reboot to bootloader (not fastboot), check with `fastboot devices` then `fastboot boot boot.img`
