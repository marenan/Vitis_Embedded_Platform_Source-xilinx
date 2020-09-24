# StreamBlocks Platforms
# Vitis Base Platform for the zcu104 Board

A basic platform targeting the ZCU106 evaluation board, which includes 2GB of DDR4 for the Processing System, GEM, USB, SDIO card interface and UART of the Processing system. It reserves most of the PL resources for user to add acceleration kernels.

# Building the Platform

**Last Tested Vivado Release: 2020.1**

The platform build process is entirely scripted. Note that as this platform
build process involves cross-compiling Linux, build of the platform is supported
on Linux environments **only** (although it is possible to build inside a VM or
Docker container).

Also note that the default PetaLinux configuration uses local scratchpad areas (/tmp). This
will **not** work if you are building on a networked file system; Yocto will error out.
Please update PetaLinux config (`petalinux-config`) to change the TMPDIR Location (CONFIG_TMP_DIR_LOCATION) to a locally-mounted hard drive if necessary.

After cloning the platform source, and with both Vivado and PetaLinux set up, run
`make` from the top-level platform directory.

By default this Makefile will install the platform to "platform_repo/xilinx_zcu106_base_202010_1/export/xilinx_zcu104_base_202010_1/"


### Build Instructions

Vitis and PetaLinux environment need to be setup before building the platform.

```bash
source /tools/Xilinx/Vitis/2020.1/settings64.sh
source /tools/petalinux-v2020.1/settings.sh
```



This package comes with sources to generate the Vitis platform with these steps:

1. Generate hardware specification file (xsa) using Vivado
2. Generate software platform environment with PetaLinux
3. Generate the Vitis platform by packaging hardware and software together using XSCT tool

The following instructions assume that you have cloned this repository and cd to the root of this platform.

Build platform from scratch:

```
make all
```

Build a platform without re-building Vivado project changes are only made in PetaLinux project.
```
make petalinux_proj XSA_DIR=<xsa dir path>
make pfm XSA_DIR=<xsa dir path>

#example:
make petalinux_proj XSA_DIR=/home/user/zcu104_base/vivado
make pfm XSA_DIR=/home/user/zcu104_base/vivado
```

From 2020.1, common Linux components are provided by Xilinx to fullfill the requirements of most evaluation requirements. Please refer to [UG1393](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/kme1569523964461.html) - Using Embedded Platforms chapter for more information. The above instructions assumes common Linux components are used so it will not generate Linux components (Kernel, rootfs) and sysroot as part of platform generation. PetaLinux project building will run for boot components generation (FSBL, PMUFW, u-boot). 

To generate Linux components according to the PetaLinux project sources in this directory, please set variable `COMMON_RFS_KRNL_SYSROOT` to `FALSE` for `make`. Example:
```bash
make all COMMON_RFS_KRNL_SYSROOT=FALSE
```

```bash
make petalinux_proj XSA_DIR=/home/user/zcu104_base/vivado COMMON_RFS_KRNL_SYSROOT=FALSE
make pfm /home/user/zcu104_base/vivado COMMON_RFS_KRNL_SYSROOT=FALSE
```

By default, the switch COMMON_RFS_KRNL_SYSROOT is set to TRUE in the platform sources and the generated platform will not contain the linux components: kernel and rootfs. User need to use these components from petalinux generated common images as mentioned in [UG1393](https://www.xilinx.com/html_docs/xilinx2020_1/vitis_doc/kme1569523964461.html)

### Build and Install the Yocto SDK

Yocto SDK (sysroot) is needed by Vitis to cross compile the applications on target platforms.

The bundled Yocto SDK "sysroot" is not generated by `make` by default. 

This can be done with command 
```
make peta_sysroot
```
It will be installed to "platform_repo/sysroot" once the build completes.

To cross-compile against this platform from the command line, source the
`environment-setup-aarch64-xilinx-linux` script to set up your environment (cross
compiler, build tools, libraries, etc).

## Platform Specification

### General Information

| Type              | Value                       |
| ----------------- | --------------------------- |
| Release Version   | xilinx_zcu104_base_202010_1 |
| Vitis version     | 2020.1                      |
| PetaLinux version | 2020.1                      |
| XRT Tag version   | 202010.2.6.655              |
| Target board      | ZCU106                      |

### Interfaces

| Interface | Region | Details            |
| --------- | ------ | ------------------ |
| UART      | PS     |                    |
| GEM       | PS     |                    |
| USB       | PS     | USB driver enabled |
| SDIO      | PS     |                    |

### Hardware Configurations

| Configuration                 | Values                                                | Details                               |
| ----------------------------- | ----------------------------------------------------- | ------------------------------------- |
| PS DDR Size                   | 4GB                                                   |                                       |
| Clocks for Kernels            | 150MHz, 300MHz, 75MHz, 100MHz, 200MHz, 400MHz, 600MHz | Clocks are synchronous to each other. |
| PS DDR Interfaces for Kernels | HP0, HP1, HP2, HP3, HPC0, HPC1                        | They share the same PS DDR.           |
| Interrupts                    | 32 Interrupts are enabled from PL Kernel to PS        |                                       |

### Software Configurations

The software configurations are based on [ZCU104 BSP](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools.html). Here is the list of additional configurations.

| Configuration                    | Values                                                       | Details |
| -------------------------------- | ------------------------------------------------------------ | ------- |
| Additional Kernel Configurations | CONFIG_CONSOLE_LOGLEVEL_DEFAULT=1                            |         |
| Additional RootFS Components     | DNF<br />e2fsprogs-resize2fs<br />parted<br />libmali-xlnx: disabled<br />xrt, xrt-dev and zocl<br />opencl-clhpp<br />opencl-headers<br />libdrm, libdrm-tests and libdrm-kms<br />packagegroup-petalinux-audio<br />packagegroup-petalinux-gstreamer<br />packagegroup-petalinux-matchbox<br />packagegroup-petalinux-opencv<br />packagegroup-petalinux-v4lutils<br />packagegroup-petalinux-vitisai<br />packagegroup-petalinux-x11<br />imagefeature-package-management<br />auto-login |         |
| Device Tree Modifications        | Add zocl node for XRT<br />Disable default dtg generated axi intc PL node and add the custom node instead |         |