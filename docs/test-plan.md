# Small Office LAN – Test Plan

This document describes the configuration checklist and verification tests for the **Small Office LAN** GNS3 lab.

---

## 1. Objectives

- Segment a small government office LAN into four VLANs (Admin, Finance, HR, IT).
- Provide inter-VLAN routing and DHCP services from a central router.
- Connect two access switches with an LACP EtherChannel trunk carrying all user VLANs.
- Secure user access ports using sticky MAC port security.
- Validate connectivity, redundancy, and basic security behavior.

---

## 2. Devices and VLANs

**Devices**

- R1 – Router-on-a-stick with DHCP pools
- 1stFloorSW – Access switch (Admin & Finance)
- 2ndFloorSW – Access switch (HR & IT)
- Admin-PC, Finance-PC, HR-PC, IT-PC – end hosts

**VLANs and subnets**

- VLAN 10 – Admin – `192.168.10.0/24` – GW `192.168.10.1`
- VLAN 20 – Finance – `192.168.20.0/24` – GW `192.168.20.1`
- VLAN 30 – HR – `192.168.30.0/24` – GW `192.168.30.1`
- VLAN 40 – IT – `192.168.40.0/24` – GW `192.168.40.1`

---

## 3. Configuration checklist

### 3.1 Router R1 – Inter-VLAN routing and DHCP

**Interfaces**

- [ ] `FastEthernet0/0` is up and has subinterfaces:
  - [ ] `Fa0/0.10` – `encapsulation dot1Q 10` – `ip address 192.168.10.1 255.255.255.0`
  - [ ] `Fa0/0.20` – `encapsulation dot1Q 20` – `ip address 192.168.20.1 255.255.255.0`
  - [ ] `Fa0/0.30` – `encapsulation dot1Q 30` – `ip address 192.168.30.1 255.255.255.0`
  - [ ] `Fa0/0.40` – `encapsulation dot1Q 40` – `ip address 192.168.40.1 255.255.255.0`

**DHCP**

- [ ] Excluded addresses:
  - [ ] `ip dhcp excluded-address 192.168.10.1 192.168.10.10`
  - [ ] `ip dhcp excluded-address 192.168.20.1 192.168.20.10`
  - [ ] `ip dhcp excluded-address 192.168.30.1 192.168.30.10`
  - [ ] `ip dhcp excluded-address 192.168.40.1 192.168.40.10`
- [ ] DHCP pools:
  - [ ] `Admin` – `network 192.168.10.0 255.255.255.0`, `default-router 192.168.10.1`, DNS 8.8.8.8
  - [ ] `Finance` – `network 192.168.20.0 255.255.255.0`, `default-router 192.168.20.1`, DNS 8.8.8.8
  - [ ] `HR` – `network 192.168.30.0 255.255.255.0`, `default-router 192.168.30.1`, DNS 8.8.8.8
  - [ ] `IT` – `network 192.168.40.0 255.255.255.0`, `default-router 192.168.40.1`, DNS 8.8.8.8

**Verification commands**

- [ ] `show ip interface brief` – all subinterfaces up/up with correct IPs
- [ ] `show ip route` – four connected routes for 192.168.10/20/30/40
- [ ] `show ip dhcp binding` – DHCP bindings for all four PCs

---

### 3.2 1stFloorSW – VLANs, trunks, EtherChannel, access ports

**VLANs**

- [ ] VLANs 10, 20, 30, 40 exist (`show vlan brief`)

**Trunk to R1**

- [ ] `Ethernet0/0`:
  - [ ] `switchport trunk encapsulation dot1q`
  - [ ] `switchport mode trunk`
  - [ ] `switchport trunk allowed vlan 10,20,30,40`

**EtherChannel to 2ndFloorSW**

- [ ] `Ethernet0/1` and `Ethernet0/2`:
  - [ ] `switchport trunk encapsulation dot1q`
  - [ ] `switchport mode trunk`
  - [ ] `switchport trunk native vlan 99`
  - [ ] `switchport trunk allowed vlan 10,20,30,40`
  - [ ] `channel-group 1 mode active`
- [ ] `Port-channel1`:
  - [ ] `switchport mode trunk`
  - [ ] `switchport trunk allowed vlan 10,20,30,40`
  - [ ] `switchport trunk native vlan 99`

**Access ports**

- [ ] `Ethernet1/0` – access VLAN 10 (Admin) with sticky port security
- [ ] `Ethernet1/1` – access VLAN 20 (Finance) with sticky port security

**Verification**

- [ ] `show interfaces trunk` – `Et0/0` and `Po1` listed, VLANs 10–40 allowed
- [ ] `show etherchannel summary` – `Po1` in use, member ports bundled
- [ ] `show port-security interface e1/0` and `e1/1` – secure-up with one sticky MAC

---

### 3.3 2ndFloorSW – VLANs, EtherChannel, access ports

**VLANs**

- [ ] VLANs 10, 20, 30, 40 exist (`show vlan brief`)

**EtherChannel back to 1stFloorSW**

- [ ] `Ethernet0/1` and `Ethernet0/2`:
  - [ ] Trunk, native VLAN 99, allowed VLANs 10–40
  - [ ] `channel-group 1 mode active`
- [ ] `Port-channel1`:
  - [ ] Trunk, allowed VLANs 10–40, native VLAN 99

**Access ports**

- [ ] `Ethernet1/0` – access VLAN 30 (HR) with sticky port security
- [ ] `Ethernet1/1` – access VLAN 40 (IT) with sticky port security

**Verification**

- [ ] `show interfaces trunk` – `Po1` listed, VLANs 10–40 allowed
- [ ] `show etherchannel summary` – `Po1` up and bundled
- [ ] `show port-security interface e1/0` and `e1/1` – secure-up with one sticky MAC

---

## 4. Functional tests

### 4.1 DHCP tests

For each PC:

- [ ] Admin-PC receives address `192.168.10.x/24` with gateway `192.168.10.1`
- [ ] Finance-PC receives `192.168.20.x/24` with gateway `192.168.20.1`
- [ ] HR-PC receives `192.168.30.x/24` with gateway `192.168.30.1`
- [ ] IT-PC receives `192.168.40.x/24` with gateway `192.168.40.1`
- [ ] `show ip dhcp binding` on R1 shows all four leases

### 4.2 Default gateway tests

- [ ] From each PC, ping its default gateway
  - Admin-PC → 192.168.10.1
  - Finance-PC → 192.168.20.1
  - HR-PC → 192.168.30.1
  - IT-PC → 192.168.40.1

### 4.3 Inter-VLAN connectivity tests

Run and record results (success/fail):

- [ ] Admin-PC ↔ Finance-PC
- [ ] Admin-PC ↔ HR-PC
- [ ] Admin-PC ↔ IT-PC
- [ ] Finance-PC ↔ HR-PC
- [ ] HR-PC ↔ IT-PC

From R1:

- [ ] `ping` each PC IP address – all reachable

---

## 5. Failure and recovery tests

### 5.1 EtherChannel redundancy

1. Shut down one EtherChannel member on 1stFloorSW:
   - `interface e0/1`  
     `shutdown`
2. Verify:
   - [ ] `show etherchannel summary` – `Po1` still up, one member active
   - [ ] Inter-VLAN pings between Admin-PC and IT-PC still succeed
3. Shut down the second member or `Po1`:
   - [ ] `Po1` goes down and 2nd floor PCs lose connectivity
4. Restore both links (`no shutdown`) and confirm full connectivity returns.

### 5.2 Port security behavior

1. On one access port (for example HR-PC on `e1/0`), trigger a violation:
   - Change the MAC or attach another VPCS to the same port.
2. Verify:
   - [ ] `show port-security interface e1/0` – violation counter increased and port in error state (depending on mode)
3. Recover:
   - [ ] `shutdown` / `no shutdown` on the interface
   - [ ] Confirm the legitimate host can reconnect.

---

## 6. Optional enhancements

Use this section for extra experiments:

- Make 1stFloorSW the STP root for VLANs 10–40.
- Add a dedicated management VLAN 99 with switch SVIs and management access.
- Add inter-VLAN ACLs on R1 (for example, deny IT → Finance but allow other flows) and re-run tests.

Document any changes and additional test results below them.
