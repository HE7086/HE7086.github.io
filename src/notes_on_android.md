# Notes on Android

Here are some notes for me when messing around with android phones.

## General

### get boot.img
```shell
adb shell su -c "dd if=/dev/block/bootdevice/by-name/boot_a" | dd of=boot_a.img
```

### extract ota zip
```shell
unzip f3f0db09cdde9dcd118da68821a445af7b0963cc.zip
payload-dumper-go payload.bin
```

### mount the extracted system image
```shell
mkdir -p mnt
simg2img system.img system.ext4
mount -t ext4 -o loop system.ext4 mnt
```

### get build.prop
```shell
adb shell su -c "cat /system/build.prop"
```

## Nothing Phone (2) Specific Notes

### get ota log
```shell
for file in $(adb shell su -c "ls /mnt/product/nt_log/ota_log/"); do
    adb shell su -c "cat /mnt/product/nt_log/ota_log/$file" > $file
done
find . -name "*.tar.gz" -exec tar xf {} \;
```
example error: (boot_a hash mismatch)
```
......
12-25 17:32:09.050  2056  2056 I update_engine: [INFO:delta_performer.cc(269)] PartitionInfo old boot sha256: 9A338489AD6B5AA0103F1F198480749211DBCBD01D879BCFBA41D900F1924FB4 size: 100663296
12-25 17:32:09.050  2056  2056 I update_engine: [INFO:delta_performer.cc(269)] PartitionInfo new boot sha256: 27B728C1A1F076C91D5E10B0BF4285E3FCF6B0527DC6AC38F2168599B49E3ABB size: 100663296
......
12-25 17:26:17.209  2041  2041 E update_engine: [ERROR:verified_source_fd.cc(50)] Unable to open ECC source partition /dev/block/bootdevice/by-name/boot_a: Success (0)
12-25 17:26:17.210  2041  2041 E update_engine: [ERROR:partition_writer.cc(317)] The hash of the source data on disk for this operation doesn't match the expected value. This could mean that the delta update payload was targeted for another version, or that the source partition was modified after it was installed, for example, by mounting a filesystem.
12-25 17:26:17.212  2041  2041 E update_engine: [ERROR:partition_writer.cc(322)] Expected:   sha256|hex = 134DBAD404F77F97FAC8EB764200375349F1916A9FDC48E7CE0D43CDB999ACAD
12-25 17:26:17.213  2041  2041 E update_engine: [ERROR:partition_writer.cc(325)] Calculated: sha256|hex = 955240FBDF80E35B3510642D70C3AB3F9F462699C4910061D3A0E7876BF1E222
12-25 17:26:17.214  2041  2041 E update_engine: [ERROR:partition_writer.cc(336)] Operation source (offset:size) in blocks: 0:1,11742:1,24575:1
12-25 17:26:17.216  2041  2041 E update_engine: [ERROR:partition_writer.cc(261)] source_fd != nullptr failed.
......
```
This error message means that the content in `boot_a` is modified.
* the expected sha256 of the partition *before update* is `old boot sha256`
* the expected sha256 of the partition *after update* is `new boot sha256`
* the other sha256s are for the generated patches

This can be fixed by flash stock boot.img (or any corresponding partition)
```shell
adb reboot bootloader
fastboot flash boot_a boot.img
```

### get stock boot.img

```shell
for tag in $(gh api repos/arter97/nothing_archive/releases | jq -r '.[].tag_name'); do
    mkdir -p "$tag"
    pushd "$tag"
    # gh release download "$tag" -R arter97/nothing_archive -p 'boot.img.zip' -O- | bsdtar -xf-
    gh release download "$tag" -R arter97/nothing_archive -p 'Pong_*-image.7z'
    bsdtar -xf Pong_*-image.7z boot.img
    popd
    sha256sum "$tag/boot.img"
done
```
Or, just the latest one
```shell
# gh release download -R arter97/nothing_archive -p 'boot.img.zip'
gh release download -R arter97/nothing_archive -p 'Pong_*-image.7z'
bsdtar -xf Pong_*-image.7z boot.img
```

### modify default url for testing network availability
```shell
# run in adb or termux root shell
# get current url, null means unchanged
settings get global captive_portal_http_url
settings get global captive_portal_https_url

# set url to e.g. miui.com
settings put global captive_portal_http_url http://connect.rom.miui.com/generate_204
settings put global captive_portal_https_url http://connect.rom.miui.com/generate_204

# reset to default
settings delete global captive_portal_http_url
settings delete global captive_portal_https_url
```
