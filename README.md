# Qubes 4.1 on a ThinkPad X1 Extreme Gen 3

This is my personal repository with files and notes
which helps to install/run Qubes 4.1 on a ThinkPad X1
Extreme Gen3

### General settings
- Disable Secure Boot
- Build Qubes 4.1 via [qubes-builder](https://github.com/QubesOS/qubes-builder) or download Qubes 4.1 alpha [ISO](http://ftp.halifax.rwth-aachen.de/qubes/iso/Qubes-R4.1.0-alpha20201014-x86_64.iso)

**Note: Qube 4.1 alpha comes with kernel 5.4.x** 

## Update to Kernel 5.9.14-1

1. Copy rpm files to dom0:
```shell
qvm-run -p ${VM} 'cat /home/user/kernel-latest-5.9.14-1.qubes.x86_64.rpm' > kernel-latest-5.9.14-1.qubes.x86_64.rpm
```
2. Install kernel RPM package:
```shell
dnf install kernel-latest-5.9.14-1.qubes.x86_64.rpm
```
3. Reboot and load the new kernel

## Fix iwlwifi load for AX200 device

Based on issue [#5615](https://github.com/QubesOS/qubes-issues/issues/5615#issuecomment-702032377)

1. Install kernel-latest-qubes-vm >= 5.8.11-3:
```shell
dnf install kernel-latest-qubes-vm-5.9.14-1.qubes.x86_64.rpm
```
2. Shutdown sys-net:
```shell
qvm-shutdown sys-net
```
3. Change Qube settings for sys-net to use the new installed kernel
```shell
qvm-prefs --set sys-net kernel "5.9.14-1"
```
4. Add `iwlwifi.disable_rxq=1` to kernel options for sys-net:
```shell
opt=$(qvm-prefs --get sys-net kernelopts)
qvm-prefs --set sys-net kernelopts "$opt iwlwifi.disable_rxq=1"
```
5. Start sys-net with the new kernel options
```shell
qvm-start sys-net
```

## Install NVIDIA driver 

Based on issue [#2526](https://github.com/QubesOS/qubes-issues/issues/2526#issuecomment-268602990):

1. Install necessary tools:
```shell
qubes-dom0-update gcc kmod grub2-tools perl-bignum make
```
2. Install kernel-devel from RPM file;
```shell
dnf install kernel-latest-devel-5.9.14-1.qubes.x86_64.rpm
```
3. Download the latest nvidia driver from https://www.nvidia.com/en-us/geforce/drivers/
   (*In my case "NVIDIA-Linux-x86_64-455.45.01.run*)
4. Copy the downloaded driver to dom0
```shell
qvm-run -p ${VM} 'cat /home/user/Downloads/NVIDIA-Linux-x86_64-455.45.01.run' > NVIDIA-Linux-x86_64-455.45.01.run
chmod +x NVIDIA-Linux-x86_64-455.45.01.run
```
5. Extract driver sources
```shell 
./NVIDIA-Linux-x86_64-455.45.01.run --ui=none --no-x-check --keep --extract-only 
```
6. Build nvidia.ko kernel driver
```shell
cd NVIDIA-*/kernel/; make module IGNORE_XEN_PRESENCE=y CC="gcc -DNV_VMAP_4_PRESENT -DNV_SIGNAL_STRUCT_RLIM"
```
7. Copy compiled driver to `/lib/modules/$(uname -r)/extra`
```shell
sudo cp nvidia.ko /lib/modules/$(uname -r)/extra/
```
8. Load the driver and check
```shell
sudo depmod -a; modinfo nvidia
```
9. Edit grub2 entry add `rd.driver.blacklist=nouveau` to the end of `GRUB_CMDLINE_LINUX`
```shell
sudo vim /etc/sysconfig/grub
```
10. Update grub.cfg for UEFI
```shell
grub2-mkconfig -o /boot/efi/EFI/qubes/grub.cfg
```
11. Disable nouveau driver; add to blacklist
```shell
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
```
12. Reboot the system and enjoy the nvidia driver

## Fix sound issues

Based on https://bugzilla.opensuse.org/show_bug.cgi?id=1176351

1. Edit grub2 entry add `snd_hda_intel.dmic_detect=0` to the end of `GRUB_CMDLINE_LINUX`
```shell
sudo vim /etc/sysconfig/grub
```
2. Update grub.cfg for UEFI
 ```shell
 grub2-mkconfig -o /boot/efi/EFI/qubes/grub.cfg
 ```
