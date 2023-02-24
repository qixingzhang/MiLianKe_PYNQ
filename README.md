# MiLianKe_PYNQ
Intructions on building PYNQ 3.0.1 image for MiLianKe 8X2CG

## Toolchain Version
* Ubuntu 18.04/20.04
* Vitis 2022.1
* PetaLinux 2022.1

## Steps
> The bitstream and PetaLinux BSP are provided, follow the optional steps to rebuild them.
### 1. Build Vivado project (Optional)
* Launch Vivado
    ```bash
    source /tools/Xilinx/Vitis/2022.1/settings64.sh
    vivado
    ```
* Create a new project with part `xczu2cg-sfvc784-1-e`
* Create block design using the provided TCL script.
    ```bash
    source system.tcl
    ```
* Generate bitstream and export hardware. Rename the bitstream and hardware description with `system.bit` and `MLK-8X2CG.xsa`

### 2. Create PetaLinux BSP (Optional)
* Source the environment
    ```bash
    source <petalinux install path>/settings.sh
    ```
* Create PetaLinux project
    ```bash
    petalinux-create -t project --template zynqMP --name MLK-8X2CG
    ```
* Config the project using hardware description
    ```bash
    cd MLK-8X2CG
    petalinux-config --get-hw-description MLK-8X2CG.xsa
    ```
    Save and Exit the configuration menu.

* Modify the device-tree

    Open the file `project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi` and replace with bellow content.
    ```c
    /include/ "system-conf.dtsi"
    / {
    };

    &sdhci1 {
        no-1-8-v;
        disable-wp;
    }
    ```

* Package the BSP
    ```bash
    cd ..
    petalinux-package --bsp -p MLK-8X2CG --output MLK-8X2CG.bsp
    ```

### 3. Build PYNQ image
* Clone PYNQ repository from github
    ```bash
    git clone https://github.com/Xilinx/PYNQ
    cd PYNQ
    git checkout image_v3.0.1
    ```

* Setup environment for new host machine
    ```
    sudo sdbuild/scripts/setup_host.sh
    ```

* Modify the device-tree of `meta-pynq`

    * Open the file `sdbuild/boot/meta-pynq/recipes-bsp/device-tree/files/pynq_bootargs.dtsi`
    * In line 3, change `root=/dev/mmcblk0p2` to `root=/dev/mmcblk1p2`

* Prepare board files and prebuilt image
    * Copy the folder `MLK-8X2CG` to `boards/`
        > You can replace the bitstream and BSP with your own one.
    * Download the RootFS: [https://bit.ly/pynq_aarch64_v3_0_1](https://bit.ly/pynq_aarch64_v3_0_1)
    * Download the SDIST: [https://bit.ly/pynq_sdist_v3_0_1](https://bit.ly/pynq_sdist_v3_0_1)

* Run make. Replace the field `<path>` with the real path of RootFS and SDIST.
    ```bash
    cd sdbuild
    make BOARDS=MLK-8X2CG PYNQ_ROOTFS=<path>/jammy.aarch64.3.0.1.tar.gz PYNQ_SDIST=<path>/pynq-3.0.1.tar.gz
    ```
    When finish, the SD card image `MLK-8X2CG-3.0.1.img` is located in `output` folder.

### 4. Boot the board
* Burn the image to a micro SD card.
* connect the PSUART to PC using a Mini USB cable. Power on the board.
* Start a serial communication program on host PC with baud rate 115200.
* Power off the board.
* Plug the SD card and set the boot mode of the board to SD boot.
* Power on the board.
* You should see a boot messages similar to the following message on the serial console. A whole example boot log is provided in `boot_log.txt`
    ```
    Xilinx Zynq MP First Stage Boot Loader 
    Release 2022.1   Apr 11 2022  -  09:29:50
    NOTICE:  BL31: v2.6(release):xlnx_rebase_v2.6_2022.1_update3
    NOTICE:  BL31: Built : 03:46:40, Mar 24 2022


    U-Boot 2022.01 (Apr 04 2022 - 07:53:54 +0000)

    CPU:   ZynqMP
    Silicon: v3
    Board: Xilinx ZynqMP
    DRAM:  1023 MiB
    PMUFW:	v1.1
    EL Level:	EL2
    Chip ID:	zu2cg
    NAND:  0 MiB
    MMC:   mmc@ff160000: 0, mmc@ff170000: 1
    Loading Environment from FAT... *** Error - No Valid Environment Area found
    *** Warning - bad env area, using default environment

    In:    serial
    Out:   serial
    Err:   serial
    Bootmode: SD_MODE1
    Reset reason:	EXTERNAL 
    Net:   
    ZYNQ GEM: ff0e0000, mdio bus ff0e0000, phyaddr -1, interface rgmii-id

    Warning: ethernet@ff0e0000 (eth0) using random MAC address - ea:89:11:c2:98:6f
    eth0: ethernet@ff0e0000
    scanning bus for devices...
    Hit any key to stop autoboot:  0 
    switch to partitions #0, OK
    mmc1 is current device
    Scanning mmc 1:1...
    Found U-Boot script /boot.scr
    2777 bytes read in 9 ms (300.8 KiB/s)
    ## Executing script at 20000000
    Trying to load boot images from mmc1
    23187200 bytes read in 1519 ms (14.6 MiB/s)
    ## Loading kernel from FIT Image at 10000000 ...
    Using 'conf-1' configuration
    Trying 'kernel-0' kernel subimage
        Description:  Linux Kernel
        Created:      2023-02-03   9:29:13 UTC
        Type:         Kernel Image
        Compression:  uncompressed
        Data Start:   0x100000d4
        Data Size:    23147008 Bytes = 22.1 MiB
        Architecture: AArch64
        OS:           Linux
        Load Address: 0x00080000
        Entry Point:  0x00080000
        Hash algo:    sha1
        Hash value:   59f220f3e2470fe145abaaccdfea913faf9d9de7
    Verifying Hash Integrity ... sha1+ OK
    ## Loading fdt from FIT Image at 10000000 ...
    Using 'conf-1' configuration
    Trying 'fdt-0' fdt subimage
        Description:  Flattened Device Tree blob
        Created:      2023-02-03   9:29:13 UTC
        Type:         Flat Device Tree
        Compression:  uncompressed
        Data Start:   0x116133cc
        Data Size:    38367 Bytes = 37.5 KiB
        Architecture: AArch64
        Hash algo:    sha1
        Hash value:   345ae7f7d6c499f86164efe152a9811556045ee7
    Verifying Hash Integrity ... sha1+ OK
    Booting using the fdt blob at 0x116133cc
    Loading Kernel Image
    Loading Device Tree to 000000003fef3000, end 000000003feff5de ... OK

    Starting kernel ...
    ```
