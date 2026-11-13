Building LTP for Android (complete walkthrough)
===============================================
This tutorial will walk you through building LTP for Android, starting from scratch.

Install build tools
-------------------
Follow the instructions here based on your operating system:

https://source.android.com/docs/setup/start/initializing?hl=en

Install repo
------------
```
sudo apt-get update
sudo apt-get install repo
```
(https://source.android.com/docs/setup/download?hl=en)

Checkout Android Platform Source
--------------------------------
**WARNING**: this step downloads ~300GB of files.

For a public AOSP checkout (see https://source.android.com/docs/setup/download):
```
mkdir android-main
cd android-main
repo init --partial-clone --no-use-superproject -b android-latest-release -u https://android.googlesource.com/platform/manifest
repo sync -c -j8
```
*(Note: In an internal Android platform checkout on `main`, use the `trunk_staging` lunch release config shown below; on a public `android-latest-release` checkout, replace `trunk_staging` with `aosp_current`, e.g., `lunch aosp_cf_x86_64_only_phone-aosp_current-userdebug`.)*

Set up environment
------------------
This configures the tooling to target Cuttlefish arm64 or x86_64:
```
source build/envsetup.sh
lunch aosp_cf_arm64_phone-trunk_staging-userdebug
```
Alternatively, to target x86_64 Cuttlefish:
```
source build/envsetup.sh
lunch aosp_cf_x86_64_phone-trunk_staging-userdebug
```
If you are targeting something else, you can follow the link below for further instructions on how to adjust the target accordingly:

https://source.android.com/docs/setup/build/building?hl=en#choose-a-target

Build ADB and atest
-------------------
```
m -j adb atest
adb version
```
(https://source.android.com/docs/setup/build/adb?hl=en)

Connect the device
------------------
Connect the device to your system and verify that ADB can see it.
```
adb devices
```

Two ways to run LTP:
--------------------
### Use atest tool to build and run tests
See [README.md](README.md#running-ltp-through-atest):
```
atest -a vts_ltp_test_arm     # 32-bit arm tests
atest -a vts_ltp_test_arm_64  # 64-bit arm tests
atest vts_ltp_test_x86_64:syscalls.chdir01_64bit  # single 64-bit x86_64 test
```

### Manually build and run tests
This is faster to build when iterating on a single test binary.

Setup temporary directories on the device:
```
adb root
adb shell "mkdir -p /data/local/tmp/ltp/tmp/ltptemp /data/local/tmp/ltp/tmp/tmpbase /data/local/tmp/ltp/tmp/tmpdir; chmod -R 777 /data/local/tmp/ltp/tmp; restorecon -F -R /data/local/tmp/ltp"
```

This example builds the `pcrypt_aead01` module (or all of `external/ltp` via `mma`), syncs it, and runs the binary:
```
m ltp_pcrypt_aead01
adb sync data
adb shell "LTPROOT=/data/local/tmp/ltp LTP_DEV_FS_TYPE=ext4 TMP=/data/local/tmp/ltp/tmp LTPTMP=/data/local/tmp/ltp/tmp/ltptemp TMPBASE=/data/local/tmp/ltp/tmp/tmpbase TMPDIR=/data/local/tmp/ltp/tmp/tmpdir LTP_TIMEOUT_MUL=2 PATH=/system/bin:/data/local/tmp/ltp:/data/nativetest64/vts_ltp_tests/testcases/bin pcrypt_aead01"
```
See [README.md](README.md#running-ltp-directly) for more details.

Modify an existing test and rerun it
------------------------------------
After making code changes to an existing test, following either of the previous steps will rebuild and run it.

If you are applying a patch file, you may do the following, filling in the patch file and binary:
```
cd external/ltp
git apply <patch filename>.patch
m ltp_<test binary>
adb sync data
adb shell "LTPROOT=/data/local/tmp/ltp LTP_DEV_FS_TYPE=ext4 TMP=/data/local/tmp/ltp/tmp LTPTMP=/data/local/tmp/ltp/tmp/ltptemp TMPBASE=/data/local/tmp/ltp/tmp/tmpbase TMPDIR=/data/local/tmp/ltp/tmp/tmpdir LTP_TIMEOUT_MUL=2 PATH=/system/bin:/data/local/tmp/ltp:/data/nativetest64/vts_ltp_tests/testcases/bin <test binary>"
```
