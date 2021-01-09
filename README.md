# Qubes 4.1 on a ThinkPad X1 Extreme Gen 3

This is my personal repository with files and notes
which helps to install/run Qubes 4.1 on a ThinkPad X1
Extreme Gen3

## Update to Kernel 5.9.14-1

1. Copy rpm files to dom0
	**dom0:** ``qvm-run -p ${VM} 'cat /home/user/kernel-latest-5.9.14-1.qubes.x86_64.rpm' > kernel-latest-5.9.14-1.qubes.x86_64.rpm``
2. Install kernel rpm package
	**dom0:** ``dnf install kernel-latest-5.9.14-1.qubes.x86_64.rpm``
3. Reboot and load the new kernel

