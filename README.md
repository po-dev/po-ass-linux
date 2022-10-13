# po-ass-linux
Linux geared towards the Orange Pi Zero 2 and other similar very-low-cost Cortex-A53 hardware.

Currently trying to get Xen hypervisor to work on the OPi2Z, then combine 4 cores and launch Jammy Ubuntu, then see if Quake 3 runs at 120 fps stable at 800x600.

Get a 3.3v TTL dongle, saves a lot of headache. Later on you can communicate to UART over Ethernet when the image you're making is close to finalized but for now that dongle helps a lot.

Get the Orange Pi Z2 latest Jammy image, burn to microsd

Follow https://vadion.com/getting-started-with-xen-hypervisor-on-orangepi-pc2-allwinner-h5/ but when downloading deps get gcc-12-arm-linux-gnueabihf  instead of the gcc-5 package.

In boot, make a directory called orangepiz2, and copy the Image* file to that folder, with just name "Image".
Copy the right dtb to the folder as well.
Finally, copy the xen binary you made to the folder and rename it xen.efi.

This boot.scr boots Xen: 

setenv kernel_addr_r  0x40080000

setenv fdt_addr       0x44000000

setenv xen_addr_r     0x45000000

setenv fdt_high       0xffffffff

ext4load mmc 0 ${kernel_addr_r} orangepiz2/Image;

ext4load mmc 0 ${fdt_addr} orangepiz2/sun50i-h616-orangepi-zero2.dtb;

ext4load mmc 0 ${xen_addr_r} orangepiz2/xen.efi;


fdt addr ${fdt_addr} 0x40000

fdt resize

fdt chosen

fdt set /chosen \#address-cells <1>

fdt set /chosen \#size-cells <1>

fdt mknod /chosen module@0

fdt set /chosen/module@0 compatible "xen,linux-zimage" "xen,multiboot-module"

fdt set /chosen/module@0 reg <${kernel_addr_r} 0x02000000>

fdt set /chosen/module@0 bootargs "console=hvc0 ro root=/dev/mmcblk0p1 rootwait clk_ignore_unused"


setenv bootargs "console=dtuart dtuart=/soc/serial@5000000 root=/dev/mmcblk0p1 rootwait dom0_mem=128M"

booti (dollar sign){xen_addr_r} - ${fdt_addr}

replace (dollar sign) with a $, formatting's borked

Gives a "Could not set up DOM0 guest OS" error here, but that's due to a lack of further setup.
