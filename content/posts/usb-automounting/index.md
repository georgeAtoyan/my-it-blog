---
date: '2026-06-06'
draft: false
title: "USB Automounting with systemd"

tags: ["linux", "SysAdmin", "systemd", "mounting"]
---

## Objective
Setting up automatic USB drive mounting on a Linux server using systemd mount and automount units

---

## Environment
- OS: Ubuntu
- Tools: system, USB drive

---

## Step 1 - Find Partition UUID

Prerequisite: Plugin USB drive

```bash
blkid /dev/sdb1
```

![sdb1](sdb1.png)

Note: UUID and filesystem type

## Step 2 - Create the Mount Unit

```bash
sudo nano /etc/systemd/system/mnt-usb.mount
```

![mnt-usb.mount](mnt-usb_mount.png)

## Step 3 - Create Automount Unit

```bash
sudo nano /etc/systemd/system/mnt-usb.automount
```

![mnt-usb.automount](mnt-usb_automount.png)

## Step 4 - Enable and Start

```bash
sudo systemctl enable mnt-usb.automount
sudo systemctl start mnt-usb.automount
```

## Step 5 - Verify and test

Check the automount is active:

```bash
systemctl status mnt-usb.automount
```

![status](automount-status.png)

Trigger the mount by accessing the path:

```bash
ls -l /mnt/usb
```

![usb_mount](usb_mount.png)

Verify it's mounted:

```bash
lsblk 
mount | grep usb
```

![verify](verify.png)