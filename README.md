# Ninkear-T40-linux-touchscreen-fix
Quick fix of the ninkear t40 firmware to enable touchscreen support

# FIX:

There's a config line in the firmware that initialise a pin to output mode instead of input which prevent the kernel to init the touchscreen, deleting that duplicate line restore the touch (and everything else still works so yeepee).

Tested on Ubuntu 26.04 LTS and Linux Mint Cinnamon


# DISCLAIMER :
If you're a competent linux dev please have a look and reach out if I did something I shouldn't.

It's provided AS IS use it at YOUR OWN RISK



```
# dump and decompile dsdt

mkdir -p /tmp/acpi_work && cd /tmp/acpi_work

sudo cat /sys/firmware/acpi/tables/DSDT > dsdt.dat

iasl -d dsdt.dat

# patching the two lines that resets the interupt/reset pin

cp dsdt.dsl dsdt_patched.dsl

sed -i '/SPC0 (0x090C0012, 0x44000201)/,+1d' dsdt_patched.dsl

# incrementing version so that it doesnt get silently ignored

sed -i 's/"A M I ", 0x01072009/"A M I ", 0x0107200A/' dsdt_patched.dsl

# recompile the patched dsl

iasl -tc dsdt_patched.dsl

# package to cpio and copy

mkdir -p ~/acpi_override/kernel/firmware/acpi

cp dsdt_patched.aml ~/acpi_override/kernel/firmware/acpi/dsdt.aml

cd ~/acpi_override

find kernel -print0 | cpio --null --create --format=newc > /tmp/acpi_override.cpio

sudo cp /tmp/acpi_override.cpio /boot/acpi_override.cpio

# tell grub to use our override and reboot

echo 'GRUB_EARLY_INITRD_LINUX_CUSTOM="acpi_override.cpio"' | sudo tee -a /etc/default/grub

sudo update-grub

# reboot and test your touchscreen
```
