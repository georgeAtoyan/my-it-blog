---
date: '2026-06-16'
draft: false
title: "DMZ Network Lab - Firewall, Routing, and nftables"

tags: ["linux", "nftables", "firewall", "sysadmin", "packet-filtering", "networking", "cybersecurity", "dmz"]
---

## Objective

The goal of this lab is to design and implement a basic DMZ (Demilitarized Zone) network using Linux virtual machines, understand routing between network segments, and apply nftables as a stateful firewall.

---

## Network Topology

```
LAN (User Network)		DMZ (Service Network)
192.168.10.0/24			192.168.20.0/24
__________________		_____________________

Client VM			Server VM
192.168.10.101			192.168.20.10
	|				|
	|				|
	________ Firewall VM ___________
		(Router + Firewall)
```

---

## VM & Routing Configuration

### Client VM

![client-config](client-conf.png)

```bash
sudo netplan try
sudo netplan apply
```

### Firewall VM (Core Router)

![firewall-config](frw-conf.png)

```bash
sudo netplan try
sudo netplan apply
```

The Firewall MUST ENABLE packet forwarding:

Temporary enable:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Permanent enable:

```bash
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-ipforward.conf
sudo sysctl --system
```

### Server VM (DMZ Host)

![server-config](srv-conf.png)

```bash
sudo netplan try
sudo netplan apply
```

---

## nftables Firewall Configuration

### Step 1 - Create table

```bash
sudo nft add table inet filter
```

### Step 2 - Create FORWARD chain (core DMZ control point)

```bash
sudo nft add chain inet filter forward \
'{ type filter hook forward priority 0; policy drop; }'
```

### Step 3 - Allow return traffic (stateful firewall behavior)

```bash
sudo nft add rule inet filter forward \
ct state established,related accept
iif enp0s8 oif enp0s9
```

### Step 4 - Allow LAN -> DMZ HTTP access (TCP 8080)

```bash
sudo nft add rule inet filter forward \
ip saddr 192.168.10.101 \
ip daddr 192.168.20.10 \
tcp dport 8080 accept
```
![nftable](nftable.png)

---

## Services Used

On Server VM:

```bash
python3 -m http.server 8080 &
```
This simulates a simple HTTP service.

![webserver](webserver.png)

---

## Testing

### LAN to DMZ connectivity (HTTP)

From Client VM:

```bash
curl http://192.168.20.10:8080
```

![curl](curl.png)

From Server VM:

![status](websrv-status.png)

### Firewall verification

Check packet flow:

```bash 
sudo tcpdump -i enp0s8
sudo tcpdump -i enp0s9
```

![enp0s8](enp0s8.png)
![enp0s9](enp0s9.png)

Expected:
- Requests visible on enp0s8
- Forwarded traffic visible on enp0s9

