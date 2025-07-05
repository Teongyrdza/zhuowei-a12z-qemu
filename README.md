# zhuowei-macos-qemu

Zhuowei macOS QEMU modified to boot without bootit.gdbscript (allows to boot other macOS versions).

## Usage instructions

- Install macOS 11.2.3
- Run [build_arm64e_kcache.sh](https://github.com/zhuowei/XNUQEMUScripts/blob/macos1101b1/macos11/build_arm64e_kcache.sh) to create a kernelcache
- Create device tree:
  - Download and extract [iPad Pro IPSW](https://updates.cdn-apple.com/2020SummerSeed/fullrestores/001-30235/6D8C0CA3-5952-4FD8-AEB3-4B4CADB626BC/iPad8,11,iPad8,12_14.0_18A5332f_Restore.ipsw)

  - Extract and modify device tree:

    ```sh
    git clone https://github.com/zhuowei/XNUQEMUScripts/
    cd XNUQEMUScripts/FourthTry
    git checkout macos1101b1
    img4tool -e -o ../../DeviceTree_iPad_Pro_iOS_14.0_b3.devicetree iPad8,11,iPad8,12_14.0_18A5332f_Restore/Firmware/all_flash/DeviceTree.j421ap.im4p
    java DTRewriter.java ../../DeviceTree_iPad_Pro_iOS_14.0_b3.devicetree ../../DeviceTree_iPad_Pro_iOS_14.0_b3_Modified.dtb
    ```

- Extract ramdisk:
  - Download and extract [macOS 11.2.3 IPSW](https://updates.cdn-apple.com/2021WinterFCS/fullrestores/071-14756/5676903C-6D55-4412-B9DF-969F15F5491A/UniversalMac_11.2.3_20D91_Restore.ipsw)
  - `img4tool -e -o ramdisk.dmg UniversalMac_11.2.3_20D91_Restore/018-17455-012.dmg`
- Build and run QEMU:

  ```sh
  git clone https://github.com/Teongyrdza/zhuowei-macos-qemu
  cd zhuowei-macos-qemu
  mkdir build && cd build
  ../configure --target-list=aarch64-softmmu
  make -j

  ./aarch64-softmmu/qemu-system-aarch64 -M virt -cpu max \
  -m 6G -d unimp,mmu -nographic -monitor stdio \
  -serial file:/dev/stdout -serial file:/dev/stdout -serial file:/dev/stdout \
  -append "-noprogress cs_enforcement_disable=1 amfi_get_out_of_my_way=1 nvram-log=1 debug=0x8 io=0xfff serial=0x7 cpus=1 rd=md0 apcie=0xffffffff" \
  -kernel /path/to/bootcache-arm64e \
  -dtb ../../DeviceTree_iPad_Pro_iOS_14.0_b3_Modified.dtb  \
  -initrd ../../ramdisk.dmg $@
  ```
