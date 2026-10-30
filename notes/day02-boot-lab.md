# Day 2: Boot lab (Ubuntu 26.04 VM, Multipass on Hyper-V)

## 1. How the VM starts
- Boot stages and their times: firmware(UEFI) -> Bootloader -> Kernel(~240ms) -> initRamfs(~1.3s) -> systemd(~2.3s) -> services(2.37s) 
- Critical chain vs blame: Critical chain provides the sequence of dependent services/processes that could lead to increased boot times. Blame provides a lis of services/processes along with theri times taken to complete once started in a sorted decreasing order. cloud-final took 15.4 s, the top of blame, but wasn’t on the critical chain, so it delayed nothing.
- Kernel command line: cat /proc/cmdline : the bootloader passes start-up parameters to the kernel, like a flash bootloader passing settings to an ECU application. The parameters are BOOT_IMAGE=/vmlinuz-xxx, root=LABEL (which disk to use as main filestystem), ro to load as read-only and console=tty1 to inform where to show the kernel messages 
- Secure Boot: Turned OFF
- 26.04 changes I verified: Dracut (Tool that builds initramfs. it replaced previous initramfs-tools), uutils got replaced wtih RUST versions, sudo-rs got replaced with RUST versions, /tmp folder is now in RAM(atmpfs) and is emptied every boot.

## 2. Installed vs running kernel
- Before and after the restart:7.0.0-34 (Before), 7.0.0-38 (After)
- First boot vs normal boot: During first boot, systemd calls cloud-init which does one-time setup: creates user, installs SSH key, enlarged the disk and set the name "lab". So, it is typically time taking. In our case it was about 8.4secs. Later, this reduced to about 4 secs 

## 3. System calls with strace
- What `cat` asked the kernel: cat triggered almost 145 system calls. About 135 were preparation, such as loading libs, setting up memory and messages etc. Only the remaining were to create and open the file
WatchdogSec=10s and NotifyAccess=all in the unit file;
the ping systemd-notify WATCHDOG=1 in the script;
what happens on timeout: SIGABRT, result watchdog;
then Restart=on-failure restarts it.
- Surprises: splice; cat and ls are one file (same inode): yes, the commands are found to be in a multicall binary. These are quite common. They are efficient wrt disk space and wrt updates, but loads several unwanted libs which might not be required for hte specific invoked command

## 4. A service supervised by systemd
- Crash → Restart=on-failure: when the service is killed or ends abruptly, systemd restarts it and the PID changes and the execution state is not saved
- Hang → only WatchdogSec catches it: when hanged, the service/process is still alive and hence the systemd doesn't restart. we need to update the unit file enabling watchdog and the program itself to notify the service. 

## Commands I used
- systemd-analyze, systemd-analyze blame, systemd-analyze critical-chain, strace -c,multipass start/stop lab,sudo dmesg, journalctl, systemctl daemon-reload, systemctl status, systemctl restart, systemctl enable, systemctl show 
uname -r, cat /proc/cmdline, mokutil --sb-state, strace -e trace=…, ls -li, type, kill -9

## What I would tell an interviewer
- Bootsequence: firmware(UEFI) -> Bootloader -> Kernel -> initRamfs -> systemd -> services
- systemd uses the service definitions to start/restart/stop the processes accordingly
- A hung process is still alive and needs watchdog enabled in the corresponding service for systemd to restart
- On my own VM, kernel 7.0.0-38 had been installed for 11 days but wasn’t running; uname -r showed the old one until I rebooted.

