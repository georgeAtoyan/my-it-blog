+++
date = '2026-04-09T16:49:31+03:00'
draft = false
title = 'Grub2 Boot Recovery: Restoring a Server at 2AM'

tags: \["Linux", "GRUB2", "Boot Recovery", "Incident Response", "SysAdmin"]

summary: "A step-by-step account of recovering a production server that dropped into the GRUB2 rescue shell at 2AM, resolved within SLA."
+++



\## 🚨 The Alert

> \*\*Time:\*\* 2:04 AM
> \*\*Alert:\*\* PageDuty - server 'web-prod-03' unreachable

> \*\*SLA:\*\* Server must be back online within 60 minutes



After attaching to the console it was immediately clear the machine had dropped into the GRUB2 shell - it never made it to the OS.



\--- 



\## 🖥️ Environment



| Detail | Value |

|---|---|

| Server | web-prod-03 |

| Root Partition | /dev/sda2 |

| Kernel Location | /boot |

| Target SLA | < 60 min |



\---



\## 🔍 What Went Wrong



The GRUB2 configuration file `/boot/grub/grub.cfg' was missing or corrupted, so the bootloader had no instructions on where to find the kernel. Instead of booting normally, the system dropped into the GRUB shell waiting for manual input.



> \*\*Lab Note:\*\* To simulate this failure, grub.cfg was intentionally corrupted

> by renaming it: `mv /boot/grub/grub.cfg /boot/grub/grub.cfg.bak'



\---



\## ⏱️ Incident Timeline



| Time | Action |

|---|---|

| 02:04 | PagerDuty alert received |

| 02:07 | Attached to console, confirmed GRUB shell |

| 02:11 | Root partition identified |

| 02:18 | Server back online via manual boot |

| 02:35 | grub.cfg regenerated, fix made permanent |



\---



\## 🛠️ Step-by-Step Recovery



\### Step 1 - List Available Block Devices



Dropped into the GRUB shell, begin by listing block devices to orient myself and identify available partitions. 



```bash

ls

```

\*\*Output:\*\* (hd0) (hd0,gpt1) (hd0,gpt2)



\--- 



\### Step 2 - Probe Partitions to Find the Linux Root Filesystem



Inspect each partition to locate valid `/boot` directory containing the kernel and initrd images.



```bash

ls (hd0,gpt2)/boot/

```



\*\*Output:\*\* vmlinuz-6.17.0.19-generic initrd.img-6.17.0-19-generic



✅ Found it - Root Filesystem is on `(hd0,gpt2)`



\---



\### Step 3 - Set Root and Prefix, Load Normal Mode



Point GRUB at the correct partition, set the module search path, then load the normal module to restore the standard boot menu.



```bash

set root=(hd0,gpt2)

set prefix=(hd0,gpt2)/boot/grub

insmod normal

normal

```



If the boot menu appears - select your kernel and boot normally.



\---

### Step 4 - Manual Boot (If Normal Mode Fails)


If Step 3 fails, boot manually by loading the kernel and initrd directly with explicit paths.



```bash

insmod Linux

linux (hd0,gpt2)/boot/vmlinuz-6.17.0-19-generic root=/dev/sda2 ro

initrd (hd0,gpt2)/boot/initrd.img-6.17.0-19-generic
boot

```



\---

### Step 5 - Regenerate GRUB Config to Prevent Recurrence



Once booted into the OS, rebuild `grub.cfg` and update the bootloader to make the fix permanent.



``bash

sudo update-grub

```



OR on RHEL/CentOS based systems:
```bash

sudo grub2-mkconfig -o /boot/grub2/grub.cfg

```



Verify the config file now exists:
```bash

ls -lh /boot/grub/grub.cfg

```



\---



\## ✅ Resolution



Server `web-prod-03` was back online at \*\*02:18\*\* - 14 min after the alert. The permanent fix was completed at 02:35, well withing the 60 min SLA.



\---



\## 📚 Lessons Learned



* Always verify `/boot/grub/grub.cfg` exists after kernel updates
* GRUB shell is recoverable - knowing these commands saves critical time
* Document your partition layout (`lsblk`) before incidents happen

