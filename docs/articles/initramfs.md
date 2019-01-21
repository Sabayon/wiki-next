# Regenerate initramfs

Sabayon uses Dracut as initramfs generator, for further customizations you can refer to the [dracut wiki](https://dracut.wiki.kernel.org/index.php/Main_Page).

## sabayon-dracut

Sabayon ships an helper tool to aid user to regenerate initramfs, `sabayon-dracut`.
You can use it to regenerate initramfs against specific kernel, or for every single one installed in your system.

To install `sabayon-dracut`, if not already present in your system, run as *root*:

    equo install sys-kernel/sabayon-dracut

Now if you would like to rebuild the initramfs, for let's say your kernel 4.18, you can just run:

    sabayon-dracut --rebuild 4.18

To rebuild the initramfs for all the kernel installed in your system:

    sabayon-dracut --rebuild-all
 
You can also check the initramfs present in your system with: `sabayon-dracut -L`.
