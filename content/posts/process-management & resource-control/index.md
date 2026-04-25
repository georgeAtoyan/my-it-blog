---
date: '2026-04-24'
draft: false
title: "Process Management & Resource Control Lab"

tags: ["Linux", "Networking", "SysAdmin", "DHCP", "Troubleshooting"]
---

# 🧪 DHCP Lab

## 📌 Objective
Simulate and troubleshoot a real-world system slowdown caused by CPU saturation and memory exhaustion.

---

## ⚙️ Environment
- Virtualization: VirtualBox
- OS: Ubuntu Server 

---

## 🛠️ Lab Setup

### Step 1 - Create CPU Load Script

```bash
nano cpu_hog.sh
chmod +x cpu_hog.sh
```

![cpu_script](cpu_heavy_script.png)

---

### Step 2 - Create Memory Load Script (Python)

```bash
nano mem_hog.py
chmod +x mem_hog.py
```

![py_script](mem_heavy_py_script.png)

---

## 🚨 Incident Simulation

### Step 1 - Check the System Load Baseline

```bash
uptime
```

![Baseline](uptime_baseline.png)

---

### Step 2 - Run the scripts

```bash
./cpu_hog.sh &
python3 mem_hog.py &
```

---

## 🔍 Investigation

### Step 1 - Check System Load

``bash
uptime
```

![Anomaly](uptime_anomaly.png)

---

### Step 2 - Monitor Processes

```bash
htop
```

![Anomaly](htop_anomaly.png)

---

### Step 3 - Identify Top Resource Consumers

```bash
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

![CPU](ps_cpu.png)

![Memory](ps_mem.png)

---

## 🧠 Analysis

### **System State**
- **CPU:** Fully saturated by multiple cpu_hog.sh processes
- **Memory:** Exhausted due to mem_hog.py
- **Swap:** Fully utilized (~2GB)
- **Kernel Action:** OOM killer terminates mem_hog.py

![OOM](kernel_log.png)

### **Key Observations***
- CPU saturation causes system lag
- Memory exhaustion triggers OOM killer
- High swap usage leads to severe performance degradation
- Multiple processes contribute to system overload

---

## ✅ Remediation

### Step 1 - Kill CPU-Intensive Processes

```bash
kill cpu-hog.sh
```

### Step 2 - Verify Memory Status

```bash
free -h
```

![Memory State](mem_state.png)

### Step 3 - Confirm Recovery

```bash
uptime
htop
```

![Baseline](back_to_baseline_uptime.png)
![Baseline](htop_back_to_baseline.png)

