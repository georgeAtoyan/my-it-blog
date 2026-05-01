---
date: '2026-05-01'
draft: false
title: "Inter-Interface Packet Forwarding Failure Investigation"

tags: ["linux", "sysadmin", "packet-forwarding", "tcp/ip", "L2", "L3", "routing", "arp", "nat", "gateway"]
---

# 🧪 Inter-Interface Packet Forwarding Failure Investigation

## 📌 Objective
Diagnose why a client cannot reach the internet

---

## ⚙️ Environment
- Virtualization: VirtualBox
- OS: ( 1 DHCP Server VM + 1 Client VM)

---

## 🛠️ Lab Network Topology

**Server (DHCP)**
- IP: 192.168.10.10
- Interfaces:
   - enp0s8 -> LAN
   - enp0s3 -> NAT

**Client VM**
- IP: 192.168.10.100
- Interface: enp0s8
- Gateway: 192.168.10.10

---

## 🚨 Incident Statement

Client cannot access the internet

---

## 🔍 PHASE 1 — Verify the Problem

Run on Client VM:

```bash
ping -c 2 192.168.10.10
ping -c 2 8.8.8.8
```
![Phase1](phase1-ping.png)

Expected outcome:
- Gateway ping -> ✅ works
- Internet ping -> ❌ fails

**Conclusion:**
- Local network works -> L2 is OK
- Internet fails -> issue beyond local network

---

## PHASE 2 — Check Client Configuration

```bash
ip a
ip route
```
![Phase2](phase2-ip.png)

**Conclusion:**
- Client is correctly configured
- Traffic is sent to gateway

---

## PHASE 3 - Verify L2

### Step 1 - Check ARP

Run on client VM:

```bash
ip neigh
```
![Phase3](phase3-step1.png)

Expected outcome:
192.168.10.10 dev enp0s8 lladdr: XX:XX:XX:XX STALE

---

### Step 2 - Capture ARP

Run on client VM:

```bash
sudo tcpdump -i enp0s8 arp
ping 192.168.10.10
```
![Phase3](phase3-step2.png)

**Conclusion:**
- ARP works -> MAC resolution works
- L2 is fully functional

---

## Phase 4 - Follow the packet

### Step 1 - Capture incoming traffic

On server:

```bash
sudo tcpdump -i enp0s8 icmp
```
On client:

```bash
ping 8.8.8.8
```

![Incoming traffic](phase4-incoming-traffic.png)

Expected outcome:
- Packets arrive at server (enp0s8)

---

Step 2 - Check outgoing traffic

On server:

```bash
sudo tcpdump -i enp0s3 icmp
```
![Outgoing traffice](phase4-outgoing-traffic.png)

Expected outcome:
- No packets leaving

---

### 🧠 Key conclusion:
Server receives traffic but does not forward it. 

---

## Phase 5 - Identify root cause

Check on server:

```bash
sysctl net.ipv4.ip_forward
```

![Phase](phase5.png)

Root Cause:
- IP forwarding disabled
- Server is NOT acting as router

---

## ✅ Remediation

### Step 1 - Enable forwarding

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

### Step 2 - Add NAT

```bash
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
```

![Phase](phase6.png)

---

### Step 3 - Testing

On client:

```bash
ping 8.8.8.8
```
![Test](test.png)

---

## 🧠 FINAL ANALYSIS

| Layer | Status |
| L2 (MAC,ARP) | ✅ working |
| L3 (routing) | ❌ broken |
| Fix | Enable forwarding |






