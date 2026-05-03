# 1stFloorSW – Small Office LAN Configuration

This document shows the step-by-step configuration for **1stFloorSW** (1st floor access switch).

Responsibilities:
- Trunk to R1 for all user VLANs
- LACP EtherChannel to 2ndFloorSW
- Access ports for Admin and Finance VLANs
- Sticky MAC port security on user ports

---

## Step 1 – Basic setup

```plaintext
enable
configure terminal

hostname 1stFloorSW
no ip domain-lookup

spanning-tree mode pvst
spanning-tree extend system-id
```

---

## Step 2 – Create VLANs

Explicitly create the user and management VLANs.

```plaintext
vlan 10
 name ADMIN
vlan 20
 name FINANCE
vlan 30
 name HR
vlan 40
 name IT
vlan 99
 name MGMT
```

Verify:

```plaintext
do show vlan brief
```

---

## Step 3 – Configure trunk to R1

FastEthernet0/0 is the trunk up to R1 carrying all user VLANs.

```plaintext
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
```

Verify later with:

```plaintext
do show interfaces trunk
```

---

## Step 4 – Configure LACP EtherChannel to 2ndFloorSW

Use Ethernet0/1 and Ethernet0/2 as member links into Port-channel1.  
Native VLAN is 99, and all user VLANs are allowed.

```plaintext
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40
 channel-group 1 mode active
 no shutdown

interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40
 channel-group 1 mode active
 no shutdown

interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40
```

Verify:

```plaintext
do show etherchannel summary
do show interfaces trunk
```

---

## Step 5 – Configure access ports for Admin and Finance

Admin‑PC on VLAN 10, Finance‑PC on VLAN 20, with sticky MAC port security.

```plaintext
interface Ethernet1/0
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security mac-address sticky
 no shutdown

interface Ethernet1/1
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security mac-address sticky
 no shutdown
```

(Sticky MAC addresses will be learned dynamically when the PCs connect.)

Verify:

```plaintext
do show port-security interface e1/0
do show port-security interface e1/1
```

---

## Step 6 – Disable unused SVI

```plaintext
interface Vlan1
 shutdown
```

---

## Step 7 – Line configuration (optional for lab)

```plaintext
line con 0
 logging synchronous
 exec-timeout 0 0

line vty 0 4
 login
 transport input telnet ssh
```

---

## Step 8 – Verification summary

After connecting R1, 2ndFloorSW, and hosts:

```plaintext
show vlan brief
show interfaces trunk
show etherchannel summary
show port-security interface e1/0
show port-security interface e1/1
```
