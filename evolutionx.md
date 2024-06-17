# Evolution X

## Add Evolution X sources

Based on https://github.com/Evolution-XYZ/manifest

```
cd
mkdir evolutionx
cd evolutionx
repo init -u https://github.com/Evolution-XYZ/manifest -b udc --git-lfs
```

## Add nabu sources

Based on

* https://github.com/Kfkcome/android_device_xiaomi_nabu device tree,
* https://github.com/Kfkcome/android_kernel_xiaomi_nabu (13.0 branch) kernel tree,
* https://github.com/Kfkcome/android_vendor_xiaomi_nabu vendor tree.

`mkdir .repo/local_manifests` then `nano .repo/local_manifests/roomservice.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <!--Device tree-->
  <project name="Kfkcome/android_device_xiaomi_nabu" path="device/xiaomi/nabu" remote="github" revision="evolution-14.0"/>
  <!--Kernel-->
  <project name="Kfkcome/android_kernel_xiaomi_nabu" path="kernel/xiaomi/nabu" remote="github" revision="14.0"/>
  <!--Vendor-->
  <project name="Kfkcome/android_vendor_xiaomi_nabu" path="vendor/xiaomi/nabu" remote="github" revision="14.0"/>
</manifest>
```

## Fetch source

`repo sync -c -j4 --force-sync --no-clone-bundle --no-tags --fail-fast`. Sometimes it will fetch too quickly data, so you will blocked for a few seconds. The repo sync will crash (`--fail-fast`), just reissue the command a few seconds later, j4 to avoid rate limiting.

## Edit build

### TLDR

#### PRODUCT_PACKAGES issue

Edit the device tree `nano device/xiaomi/nabu/lineage_nabu.mk`, to add at the end on a new line `TARGET_DISABLE_EPPE := true`.

#### Recovery SE permissive issue

Edit the device tree `nano device/xiaomi/nabu/sepolicy/private/recovery.te`, add a `#` before each line, it should look like this :

```
# Set recovery to permissive
#recovery_only(`
#  permissive recovery;
#')
```

####

`mv kernel/xiaomi/nabu/RemovePackages ../`
`mv kernel/xiaomi/nabu/rootdir ../`
`mv kernel/xiaomi/nabu/power ../`
`nano device/xiaomi/nabu/BoardConfig.mk`

mkdir -p kernel/xiaomi/nabu/arch/arm64/configs/
nano kernel/xiaomi/nabu/arch/arm64/configs/nabu_defconfig
https://github.com/Dark-Matter7232/kernel_msm-4.14-nabu/blob/a5d912c395db6bc631b07798eb045434ea9748bb/arch/arm64/configs/nabu_defconfig


`BUILD_BROKEN_INCORRECT_PARTITION_IMAGES := true`

`nano`

L34 : `RFS_APQ_GNSS_SYMLINKS := $(TARGET_OUT_VENDOR)/rfs/apq/gnss` (remove trailing slash)
L46 : `RFS_MDM_ADSP_SYMLINKS := $(TARGET_OUT_VENDOR)/rfs/mdm/adsp` (remove trailing slash)
L58 : `RFS_MDM_CDSP_SYMLINKS := $(TARGET_OUT_VENDOR)/rfs/mdm/cdsp` (remove trailing slash)
L70 : `RFS_MDM_MPSS_SYMLINKS := $(TARGET_OUT_VENDOR)/rfs/mdm/mpss` (remove trailing slash)
L82 : `RFS_MDM_SLPI_SYMLINKS := $(TARGET_OUT_VENDOR)/rfs/mdm/slpi` (remove trailing slash)
L94 : `RFS_MDM_TN_SYMLINKS := $(TARGET_OUT_VENDOR)/rfs/mdm/tn` (remove trailing slash)
L106 : `RFS_MSM_ADSP_SYMLINKS := $(TARGET_OUT_VENDOR)/rfs/msm/adsp` (remove trailing slash)
L118 : `RFS_MSM_CDSP_SYMLINKS := $(TARGET_OUT_VENDOR)/rfs/msm/cdsp` (remove trailing slash)
L130 : `RFS_MSM_MPSS_SYMLINKS := $(TARGET_OUT_VENDOR)/rfs/msm/mpss` (remove trailing slash)
L142 : `RFS_MSM_SLPI_SYMLINKS := $(TARGET_OUT_VENDOR)/rfs/msm/slpi` (remove trailing slash)

### Explaination

#### PRODUCT_PACKAGES issue

At first I had an issue.

```
build/make/core/main.mk:1376: warning:  device/xiaomi/nabu/lineage_nabu.mk includes non-existent modules in PRODUCT_PACKAGES
Offending entries:
AndroidMediaShell
CastAuthPrebuilt
DevicePersonalizationPrebuiltPixelTablet2023
DockManagerPrebuilt
DockSetup
GoogleHomePrebuilt
GoogleKeepPrebuilt
LatinIMEGooglePrebuilt2023Tablet
PixelWallpapers2023Tablet
PlayBooksPrebuilt
SCONE-v14570
SafetyHubHideApps
TipsPrebuilt
android.hardware.light-service.xiaomi
build/make/core/main.mk:1376: error: Build failed.
```

`mgrep LatinIMEGooglePrebuilt2023Tablet` show the packages are added in `vendor/gms/gms_full_tablet_wifionly.mk`. Let's dig.

When we trigger a build, we trigger the `lineage_nabu.mk`. [Source](https://github.com/Kfkcome/android_device_xiaomi_nabu/blob/bba11de87d71a4127fdc282ff021342520e17e4a/AndroidProducts.mk#L2)

`lineage_nabu.mk` refers to `vendor/lineage/config/common_full_tablet_wifionly.mk`. [Source](https://github.com/Kfkcome/android_device_xiaomi_nabu/blob/bba11de87d71a4127fdc282ff021342520e17e4a/lineage_nabu.mk#L16C25-L16C77)

To get the source, I went to [the manifest source](https://github.com/Evolution-XYZ/manifest/blob/dc802517618ed5208718e2732b0068c904c1ce03/snippets/evolution.xml#L104). We see that vendor/lineage refers to https://github.com/Evolution-XYZ/vendor_evolution.

So `vendor/lineage/config/common_full_tablet_wifionly.mk` is [this file](https://github.com/Evolution-XYZ/vendor_evolution/blob/udc/config/common_full_tablet_wifionly.mk), which refers to `vendor/gms/gms_full_tablet_wifionly.mk`. [Source](https://github.com/Evolution-XYZ/vendor_evolution/blob/672e50c61cb81cf9a7c28508f6d77a83e57049ce/config/common_full_tablet_wifionly.mk#L23)

When [I asked on evolutionx discord #builders](https://discord.com/channels/670512508871639041/1209547653566955520/1251638724597649419), user `b4579` (discord id `957037598042386442`) [told me](https://discord.com/channels/670512508871639041/1209547653566955520/1251639911950581801) to add `TARGET_DISABLE_EPPE := true`.

#### Recovery SE permissive issue

I had this issue :

```
[ 80% 145720/180633] //system/sepolicy:sepolicy.recovery Compiling cil files for sepolicy.recovery [common]
FAILED: out/soong/.intermediates/system/sepolicy/sepolicy.recovery/android_common/nabu/sepolicy
out/host/linux-x86/bin/secilc -m -M true -G -c 30 out/soong/.intermediates/system/sepolicy/recovery_sepolicy.cil/android_common/nabu/recovery_sepolicy.cil -o out/soong/.intermediates/system/sepolicy/sepolicy.recovery/android_common/nabu/sepolicy_policy -f /dev/null && out/host/linux-x86/bin/sepolicy-analyze out/soong/.intermediates/system/sepolicy/sepolicy.recovery/android_common/nabu/sepolicy_policy permissive  >  out/soong/.intermediates/system/sepolicy/sepolicy.recovery/android_common/nabu/sepolicy_permissive && if test -s out/soong/.intermediates/system/sepolicy/sepolicy.recovery/android_common/nabu/sepolicy_permissive ; then echo -e "==========\nERROR: permissive domains not allowed in user builds\nList of invalid domains:" && cat  out/soong/.intermediates/system/sepolicy/sepolicy.recovery/android_common/nabu/sepolicy_permissive ; exit 1; fi && cp -f out/soong/.intermediates/system/sepolicy/sepolicy.recovery/android_common/nabu/sepolicy_policy out/soong/.intermediates/system/sepolicy/sepolicy.recovery/android_common/nabu/sepolicy && rm -f out/soong/.intermediates/system/sepolicy/sepolicy.recovery/android_common/nabu/sepolicy_permissive out/soong/.intermediates/system/sepolicy/sepolicy.recovery/android_common/nabu/sepolicy_policy # hash of input list: 50043e84f3ecee3913b74746c62fe9d0944beabb4d44e861d5f55da9a155aba3
-e ==========
ERROR: permissive domains not allowed in user builds
List of invalid domains:
recovery
16:45:40 ninja failed with: exit status 1
```

When searching for `permissive` in the [device tree](https://github.com/search?q=repo%3AKfkcome%2Fandroid_device_xiaomi_nabu%20permissive%20&type=code), the only match is `sepolicy/private/recovery.te` : you just need to comment all the file.

#### Normalized

```
[ 99% 35912/35931] Target vendor fs image: out/target/product/nabu/vendor.img
FAILED: out/target/product/nabu/vendor.img
/bin/bash -c "(mkdir -p out/target/product/nabu/vendor ) && (mkdir -p out/target/product/nabu/obj/PACKAGING/vendor_intermediates && rm -rf out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"vendor_fs_type=ext4\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"vendor_disable_sparse=true\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"vendor_selinux_fc=out/target/product/nabu/obj/ETC/file_contexts.bin_intermediates/file_contexts.bin\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"building_vendor_image=true\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"ext_mkuserimg=mkuserimg_mke2fs\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"extfs_sparse_flag=-s\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"erofs_sparse_flag=-s\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"squashfs_sparse_flag=-s\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"f2fs_sparse_flag=-S\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"avb_avbtool=avbtool\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"avb_vendor_hashtree_enable=true\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"avb_vendor_add_hashtree_footer_args=--prop com.android.build.vendor.os_version:14 --prop com.android.build.vendor.fingerprint:\$(cat out/target/product/nabu/build_fingerprint.txt) --prop com.android.build.vendor.security_patch:2023-01-01\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"recovery_as_boot=true\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"root_dir=out/target/product/nabu/root\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"use_dynamic_partition_size=true\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (echo \"skip_fsck=true\" >>  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (sort -o  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt  out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt ) && (PATH=out/host/linux-x86/bin/:system/extras/ext4_utils/:\$PATH out/host/linux-x86/bin/build_image --input-directory-filter-file out/target/product/nabu/obj/PACKAGING/vendor_intermediates/file_list.txt out/target/product/nabu/vendor out/target/product/nabu/obj/PACKAGING/vendor_intermediates/vendor_image_info.txt out/target/product/nabu/vendor.img out/target/product/nabu/system ) && (true )"
rfs/apq/gnss/: not normalized
17:11:30 ninja failed with: exit status 1
```

Trailing slashes in device tree android.mk.

## Build

```
. build/envsetup.sh
lunch lineage_nabu-user
m evolution
```

Get the boot.img using scp, reboot to bootloader (not fastboot), check with `fastboot devices` then `fastboot boot boot.img`
