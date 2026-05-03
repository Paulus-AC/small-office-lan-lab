# R1 – Small Office LAN Configuration

This document shows the step-by-step configuration for router **R1** in the Small Office LAN GNS3 lab.

R1 provides:
- Inter-VLAN routing using router-on-a-stick
- DHCP for four VLANs (Admin, Finance, HR, IT)

---

## Step 1 – Basic setup

Enter privileged mode and basic housekeeping.

```plaintext
enable
configure terminal

hostname R1
no ip domain-lookup
ip cef
```

---

## Step 2 – Configure DHCP exclusions

Reserve the first 10 addresses in each subnet (gateway + infrastructure, printers, etc.).

```plaintext
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp excluded-address 192.168.40.1 192.168.40.10
```

---

## Step 3 – Configure DHCP pools

Create one pool per department.

```plaintext
ip dhcp pool Admin
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8

ip dhcp pool Finance
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8

ip dhcp pool HR
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8

ip dhcp pool IT
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
 dns-server 8.8.8.8
```

---

## Step 4 – Configure router‑on‑a‑stick subinterfaces

Use FastEthernet0/0 as the trunk towards 1stFloorSW, with four 802.1Q subinterfaces.

```plaintext
interface FastEthernet0/0
 no ip address
 no shutdown

interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0

interface FastEthernet0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
```

Leave FastEthernet0/1 unused:

```plaintext
interface FastEthernet0/1
 shutdown
```

---

## Step 5 – Line configuration (optional for lab)

```plaintext
line con 0
 logging synchronous
 exec-timeout 0 0

line vty 0 4
 login
 transport input telnet ssh
```

---

## Step 6 – Verification commands

After bringing up switches and PCs, verify:

```plaintext
show ip interface brief
show ip route
show ip dhcp binding
```

You should see the four subinterfaces up/up, four connected routes for the 192.168.10/20/30/40 networks, and DHCP bindings for all PCs.
