# PBRP Build

## Add PBRP sources

Based on https://github.com/PitchBlackRecoveryProject/manifest_pb

```
cd
mkdir pbrp
cd pbrp
repo init -u https://github.com/PitchBlackRecoveryProject/manifest_pb -b android-12.1 --depth=1
```

## Add nabu sources

Based on

* https://github.com/SIDDK24/android_device_xiaomi_nabu-pbrp device tree,
* https://github.com/LesGaR/android_kernel_xiaomi_nabu (13.0 branch) kernel tree

Beware : the [official device tree](https://github.com/PitchBlackRecoveryProject/android_device_xiaomi_nabu-pbrp) is not working, because they are using generic kernel and not specific one (blue screen like the Kfkcome kernel). I didn't test [LesGaR fork](https://github.com/LesGaR/android_device_xiaomi_nabu-PBRP).

Beware : most known https://github.com/Kfkcome/android_kernel_xiaomi_nabu not working, even on 13.0 branch : we have a blue screen, only display part of notification bar. The fork by LesGaR is working (on 13 branch), it has several commits about fixing recovery, fxing refresh rate in twrp (9f2eaba, da708a6) so I think that's why only this one works.

`mkdir .repo/local_manifests` then `nano .repo/local_manifests/roomservice.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <!--Device tree-->
  <project name="SIDDK24/android_device_xiaomi_nabu-pbrp" path="device/xiaomi/nabu" remote="github" revision="android-12.1"/>
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
