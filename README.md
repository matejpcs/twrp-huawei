# TWRP device tree for Huawei Y5 2018 (DRA-L21)

This is a port of the Huawei Y5 Lite 2018 Dura recovery tree for the 2 GB
Huawei Y5 2018. The target model is **DRA-L21** (2 GB RAM, 16 GB storage),
while the Android device codename remains **dura** because the recovery
sources, MT6739 platform, kernel image, and partition layout are shared by
the Dura family.

Do not use this tree as-is on the 1 GB Y5 Lite model **DRA-LX5** or on a
different Dura SKU. The bootloader model and the partition map must match the
phone being flashed.

## Device specification

|                   Basic | Spec Sheet                                                    |
| ----------------------: | :------------------------------------------------------------ |
| Device                  | Huawei Y5 2018                                               |
| Model                   | DRA-L21                                                       |
| Internal codename       | dura                                                          |
| Hardware family         | HWDRA-M                                                       |
| Chipset                 | MediaTek MT6739                                               |
| CPU                     | Quad-core 1.5 GHz Cortex-A53                                  |
| GPU                     | PowerVR GE8100                                                |
| RAM                     | 2 GB                                                          |
| Storage                 | 16 GB                                                         |
| Battery                 | Li-Ion 3020 mAh                                               |
| Dimensions              | 146.5 x 70.9 x 8.3 mm                                         |
| Display                 | LCD, 720 x 1440 pixels, 5.45 inches (~295 ppi density)        |
| Rear camera             | 8 MP, f/2.2                                                   |
| Front camera            | 5 MP, flash                                                   |
| Shipped Android Version | 8.1.0                                                         |

<img src="https://user-images.githubusercontent.com/67373913/169615300-663a14f9-cdf9-466a-9f15-3cfee98ca5c4.png" width="40%">

## Source layout

The tree is consumed from `device/huawei/dura` in an Omni TWRP checkout.

- `BoardConfig.mk` defines the 32-bit ARM MT6739 target, image sizes,
  recovery options, encryption settings, and the prebuilt kernel format.
- `omni_dura.mk` is the normal Omni product; `twrp_dura.mk` is the minimal
  TWRP product definition.
- `system.prop` supplies the DRA-L21 product identity at recovery runtime.
- `recovery.fstab` maps the Dura eMMC partitions and removable storage.
- `recovery/root/` contains the Huawei and MT6739 recovery init rules,
  USB gadget setup, permissions, and the bundled `parted` binary.
- `prebuilt/zImage-dtb` is used because this repository does not include a
  kernel source checkout.

The shared `dura` codename is intentional. The user-visible properties are
set to `HUAWEI`, `DRA-L21`, and `HWDRA-M`; changing the internal codename
would require renaming the complete Android product path and lunch target.

The target GPT was checked against a full DRA-L21 MTKClient dump. Important
image sizes are 48 MiB for both `recovery` and `erecovery`, 24 MiB for
`boot`, 1,795,162,112 bytes for `system`, 385,875,968 bytes for `vendor`,
and 11,658,051,072 bytes for `userdata`. These values are reflected in
`BoardConfig.mk`.
Keep the original `gpt.bin`, `gpt_backup.bin`, `recovery.bin`, and
`erecovery.bin` backups before testing any image.

## Build requirements

Use a Linux x86_64 host with a Java 8-compatible Android build environment.
The tree is designed for the legacy Omni TWRP 8.1 manifest. Install the
usual Android build dependencies for the selected Linux distribution,
including Git, Repo, Python, Make, GCC, OpenJDK 8, `bc`, `bison`, `flex`,
`gperf`, `libssl-dev`, `libxml2-utils`, `lzop`, and `cpio`.

Keep at least 80 GB free disk space and enough RAM/swap for a full Android
recovery build. The commands below use Bash; run them from a normal user
account rather than as root.

## Download the build tree

```bash
mkdir -p "$HOME/twrp-dra-l21"
cd "$HOME/twrp-dra-l21"

repo init \
  -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git \
  -b twrp-8.1
repo sync -c --no-clone-bundle --no-tags -j"$(nproc --all)"

mkdir -p device/huawei
git clone --branch dra-l21 --single-branch \
  https://github.com/matejpcs/twrp-huawei.git \
  device/huawei/dura
```

If the device-tree directory already exists, update it instead of cloning a
second copy:

```bash
git -C device/huawei/dura fetch origin dra-l21
git -C device/huawei/dura checkout dra-l21
git -C device/huawei/dura pull --ff-only origin dra-l21
```

## Configure and build

```bash
cd "$HOME/twrp-dra-l21"
source build/envsetup.sh
lunch omni_dura-eng
mka recoveryimage
```

The expected image is:

```text
out/target/product/dura/recovery.img
```

`omni_dura-userdebug` and `omni_dura-user` are also exposed by
`AndroidProducts.mk`, but `omni_dura-eng` is the recommended recovery build
target because it keeps ADB and debugging available while validating the
port.

For a clean rebuild after changing board or fstab settings:

```bash
make clobber
source build/envsetup.sh
lunch omni_dura-eng
mka recoveryimage
```

Do not use `make clobber` if you need to preserve unrelated build products.
For a less disruptive rebuild, remove only `out/target/product/dura` and
re-run the configuration and build commands.

## Verify before flashing

Run these checks from the repository root:

```bash
git -C device/huawei/dura status --short
git -C device/huawei/dura diff --check
test -s out/target/product/dura/recovery.img
sha256sum out/target/product/dura/recovery.img
```

Before flashing, confirm all of the following:

1. The phone identifies itself as **DRA-L21**, not DRA-LX5, DRA-LX3, or a
   different Huawei device.
2. The bootloader is unlocked and the original recovery/boot images and
   important partitions are backed up.
3. The supplied stock firmware uses the same Dura partition names as
   `recovery.fstab`.
4. The image is tested by booting it temporarily first, when the bootloader
   tooling supports that operation.

This repository contains a prebuilt kernel and no device in the build
environment is available for hardware validation. A successful compilation
does not by itself prove that the image boots on every regional DRA-L21
firmware revision.

## Troubleshooting

- **Unknown target `omni_dura-eng`:** run `source build/envsetup.sh` again
  and confirm that the tree is at `device/huawei/dura`.
- **Missing dependencies:** keep `ALLOW_MISSING_DEPENDENCIES := true`, as
  supplied by `BoardConfig.mk`, and use the matching `twrp-8.1` manifest.
- **Init module not present:** verify that the product target is `dura`.
  `Android.mk` intentionally keys the device module on `TARGET_DEVICE=dura`,
  not on the `HWDRA-M` hardware-family property.
- **Data does not decrypt or mount:** do not guess a different partition
  table. Capture the stock DRA-L21 recovery fstab and compare the userdata,
  metadata, filesystem type, and encryption source before changing the tree.
- **USB/ADB problems:** check the recovery log and confirm that the image
  was built from the included MT6739 recovery init files and the expected
  stock firmware/vendor blobs.

## Recovery feature checklist

Blocking checks
- [✅] Correct screen/recovery size
- [✅] Working Touch, screen
- [✅] Backup to internal/microSD
- [✅] Restore from internal/microSD
- [✅] reboot to system
- [✅] ADB

Medium checks
- [✅] update.zip sideload
- [✅] UI colors (red/blue inversions)
- [✅] Screen goes off and on
- [✅] F2FS/EXT4 Support, exFAT/NTFS where supported
- [✅] all important partitions listed in mount/backup lists
- [✅] backup/restore to/from external (USB-OTG) storage (not supported by the device)
- [✅] backup/restore to/from adb (https://gerrit.omnirom.org/#/c/15943/)
- [✅] decrypt /data
- [✅] Correct date

Minor checks
- [ ] MTP export
- [✅] reboot to bootloader
- [✅] reboot to recovery
- [✅] poweroff
- [✅] battery level
- [✅] temperature
- [✅] encrypted backups
- [✅] input devices via USB (USB-OTG) - keyboard, mouse and disks (not supported by the device)
- [✅] USB mass storage export
- [✅] set brightness
- [✅] vibrate
- [✅] screenshot
- [✅] partition SD card

The checklist above is retained from the original Dura tree. It is a feature
reference, not a substitute for testing this DRA-L21 build on real hardware.
