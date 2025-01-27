# Vitis Embedded Platform Source Repository

Welcome to the Vitis embedded platform source repository. This repository contains the
source code needed to recreate, modify, and extend the [Xilinx-provided Vitis embedded
platforms][3]. 

[3]: https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-platforms.html

To build the Vitis embedded platforms from source code in this repository, you will need to have the following tools installed and follow the [build instructions](#build-instructions):

- A Linux-based host OS supported by Vitis and PetaLinux
- [Vitis][1] 2021.1
- [PetaLinux][2] 2021.1 or [Commong Software Image](#installing-the-common-software) 2021.1

[1]: https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vitis.html
[2]: https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools.html

To learn how to customize Vitis embedded platforms, please refer to [Vitis Platform Creation Tutorials](https://github.com/Xilinx/Vitis-Tutorials/tree/master/Vitis_Platform_Creation).

This repository is divided into two sections:

- The **Xilinx_Official_Platforms** directory contains official platform source provided by
  Xilinx. These official platforms are fully tested with all official Vitis examples, tutorials,
  etc.

- The **Xilinx_Reference_Platforms** directory contains "unofficial" platform source. These
  platforms generally exist to illustrate particular concepts, provide specific I/O interfaces
  (e.g. MIPI and HDMI), etc. They are tested with the particular examples delivered for those
  platforms and Xilinx makes an effort to ensure compatibility, but they are not regressed
  and tested to the same degree as the official platforms.

## Build Instructions

  Vitis and PetaLinux environment need to be setup before building the platform.

  ```bash
  source <Vitis_install_path>/Vitis/2021.1/settings64.sh
  source <PetaLinux_install_path>/settings.sh
  ```
  This package comes with sources to generate the Vitis platform with these steps:

  1. Generate hardware specification file (XSA) using Vivado.
  2. Generate software components of platform (using either Petalinux or XSCT).
  3. Generate the Vitis platform by packaging hardware and software together using XSCT tool


The following table summarizes use cases of platforms build :
    
- **Pre-Synth XSA** : Hardware specification file (XSA) in these platforms will not have bitstream and generation time of XSA   is quick. By default, Vitis platform Makefile generates pre-synth platforms.
- **Post-Impl XSA** : XSA in this flow contains PL bitstream in it and generation time will be longer.

  |     | Build Platform with pre-built Linux | Build platform with petalinux |
  | --- | --- | --- |
  | *Introduction:* | This method uses pre-built software components. Time taken for building sw components can be reduced | This method builds all software components. Users can customise software components to have additional libraries, packages etc |
  | *Requirements:* | Common Software. Use Sysroot SDK from Common Software | Petalinux Tool. Install PetaLinux generated SDK for this project |
  | *Pre-Synth command:* | make all PREBUILT_LINUX_PATH=<path/to/boot_files/dir> | make all |
  | *Post-Impl command:* | make all PRE_SYNTH=FALSE PREBUILT_LINUX_PATH=<path/to/boot_files/dir> | make all PRE_SYNTH=FALSE |

The Flags used in commands of above table are explained below :

- *PREBUILT_LINUX_PATH*: Provide a pre-built Linux image to skip the PetaLinux image building process. The build flow will generate device tree from XSA and run platform package with the pre-built Linux image. Common Linux components are provided as pre-built binaries by Xilinx to fullfill most evaluation requirements. Please refer to [UG1393- Using Embedded Platforms chapter][4] for more information. Set this flag to point common software components(u-boot, boot.scr, bl31.elf). These files are architecture specific. Zynq, zynqMP and Versal have different software component files. To generate linux components locally according to the PetaLinux project sources in this directory, donot set this flag.

    ```bash
    make all PREBUILT_LINUX_PATH=<common_sw_dir> 
    # This dir should contain u-boot.elf, boot.scr and bl31.elf
    ```

[4]: https://www.xilinx.com/cgi-bin/docs/rdoc?t=vitis+doc;v=latest;d=usingembeddedplatforms.html;a=mym1591997179666

- *PRE_SYNTH*: This flag is used to specify XSA type that needs to be built. The default value is `TRUE`, which means pre-synth XSA is generated. To generate post-impl XSA, set this variable to `FALSE`. The build flow will run through implementation in Vivado and include implementation results to the post-impl XSA.

- *XSA*: This flag is used when building a platform with a pre-defined XSA. Example usage:

    ```bash
    make linux XSA=/home/user/<platform_dir>/vivado/build/<platform_name>.xsa
    make platform
    ```

Notes:

- When building PetaLinux image from source code, the build temp directory is set to **/tmp/<platform_name>**. You can update the build temp directory by modifying CONFIG_TMP_DIR_LOCATION option in **<platform_name>/sw/petalinux/project-spec/configs/config** file.

## Installing the Common Software

Xilinx provides pre-built Common Software images so that user can skip PetaLinux project creation and build process for common usages. They can be downloaded from Xilinx Download Center under [Vitis Embedded Platform][3] tab or [PetaLinux tab][2]. Please refer to *Common images for Embedded Vitis platforms* section. 

The common image packages download contains these components

- Prebuilt Linux kernel
- Prebuilt root file system
- `sdk.sh` to generate sysroot
- boot files (pre-built u-boot.elf, boot.scr, bl31.elf, etc)

You can use the following command to extract the tar image.

```bash
tar -xzf xilinx-<arch>-common-<version>.tar.gz
```

To use the common software, pass the extracted path to `PREBUILT_LINUX_PATH` parameter of make command.

## Installing the Yocto SDK

Vitis applications require to build with libaries in sysroot. Sysroot can be installed from `sdk.sh`, which is generated by PetaLinux or included in Common Software image.

To install sysroot from `sdk.sh`, you can run the script `sdk.sh` directly.

```bash
./sdk.sh
```

You can also check the available options for scripting.

```bash
./sdk.sh --help
Usage: sdk.sh [-y] [-d <dir>]
  -y         Automatic yes to all prompts
  -d <dir>   Install the SDK to <dir>
======== Extensible SDK only options ============
  -n         Do not prepare the build system
  -p         Publish mode (implies -n)
======== Advanced DEBUGGING ONLY OPTIONS ========
  -S         Save relocation scripts
  -R         Do not relocate executables
  -D         use set -x to see what is going on
  -l         list files that will be extracted
```


Running the command `make petalinux_sysroot` installs sysroot to **platform_repo/sysroot**. This requires PetaLinux tool to be sourced.

To cross-compile with this SDK from the command line, run `source ./environment-setup-*-xilinx-linux` to set up your environment (cross-compile build tools, libraries, etc).
