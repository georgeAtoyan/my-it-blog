---
date: '2026-04-19'
draft: false
title: "Log Analysis & System Debugging"

tags: ["Linux", "Debugging", "Logs", "Incident Response"]
---

# 🧪 Log Analysis & System Debugging Lab

## 📌 Objective
- Read system logs like real incidents
- Identify root causes
- Debug common system failures

---

## ⚙️ Environment 

- OS: Ubuntu
- Tools: journalctl, df, du, dmesg

---

## 🚨 Scenario 1: Service Crash Investigation

### Step 1 - Break the service

```bash
sudo nano /etc/nginx/nginx.conf
```
Add invalid line:

```bash
invalid_directive;
```

Restart:

```bash
sudo systemctl restart nginx
```

### Step 2 - Investigate

Check status:

```bash
systemctl status nginx
```

![Nginx](nginx.png)

![Failed](failed_nginx.png)

Check logs:

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

![Journal](journalctl.png)

Switch to application-level debugging:

```bash
sudo nginx -t
```

![nginx_error](nginx_error.png)

### Step 3 - Fix conf

Remove bad line "invalid_directive"

### Step 4 - Verify

```bash
sudo nginx -t
sudo systemctl restart nginx
```
![nginx_fix](nginx_fix.png)

---

## 🚨 Scenario 2: Disk Full Incident

### Step 1 - Fill disk

```bash
sudo fallocate -l 16G /bigfile
```

### Step 2 - Trigger failiure

```bash
echo "hello" > file.txt
```

🔍 Expected Result: 
No space left on device

### Step 3 - Investigate

```bash
df -h
```

![Disk_Space](disk_space.png)

Check system messages:

```bash
dmesg | grep -i space
```
![dmesg](dmesg.png)

Check logs (may be unreliable):

```bash
sudo systemctl -xe
```

![journalctl](journalctl -xe.png)

Find what filled the disk:

```bash
du -sh /* 2>/dev/null
```

![Disk_Usage](disk_usage.png)

### Step 4 - Fix the issue:

```bash
sudo rm /bigfile
df -h
```
![fix](fix.png)

### Step 5 - Verify recovery:

```bash
echo "test" > file.txt
```

![Recovery](recovery.png)

---

## 🧠 Key Takeaways

- Disk issues often look like unrelated failures
- Logs may fail or be incomplete
- Always check disk first
- Must verify fixes, not assume



