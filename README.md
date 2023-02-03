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

* Modify the Device-tree

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

* Run make
    ```bash
    cd sdbuild
    make BOARDS=MLK-8X2CG PYNQ_ROOTFS=jammy.aarch64.3.0.1.tar.gz PYNQ_SDIST=pynq-3.0.1.tar.gz
    ```
    When finish, the SD card image is located in `output` folder.