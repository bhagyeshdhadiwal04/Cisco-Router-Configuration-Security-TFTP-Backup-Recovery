# Routing Security & Backup–Recovery using TFTP Server (Cisco Packet Tracer)

**Author:** Bhagyesh
**Tool Used:** Cisco Packet Tracer
**Category:** Networking / Network Security / Router Administration

---

## 📌 1. Project Objective

This project demonstrates how to secure router-to-router communication using
routing security best practices, and how to back up and recover Cisco device
configurations (running-config, startup-config, IOS image) using a **TFTP
server**. It simulates a small enterprise network with two routers, two
switches, a TFTP server, and end devices.

---

## 🖧 2. Network Topology

```
        PC0                                          PC1
         |                                             |
      Switch0                                       Switch1
         |                                             |
      R1 (Gi0/0)-------- Serial Link --------(Gi0/0) R2
    (Gi0/1)                                       (Gi0/1)
         |                                             |
   TFTP Server                                     Server (Optional)
```

| Device        | Interface  | IP Address       | Subnet Mask      |
|---------------|-----------|-------------------|-------------------|
| R1            | Gi0/0     | 192.168.1.1       | 255.255.255.0     |
| R1            | Se0/0/0   | 10.0.0.1          | 255.255.255.252   |
| R1            | Gi0/1     | 192.168.10.1      | 255.255.255.0     |
| R2            | Gi0/0     | 192.168.2.1       | 255.255.255.0     |
| R2            | Se0/0/0   | 10.0.0.2          | 255.255.255.252   |
| PC0           | NIC       | 192.168.1.10      | 255.255.255.0     |
| PC1           | NIC       | 192.168.2.10      | 255.255.255.0     |
| TFTP Server   | NIC       | 192.168.10.100    | 255.255.255.0     |

---

## 🛠 3. Tools & Requirements

- Cisco Packet Tracer (8.x)
- 2 Routers (Type: 2911 or 1941)
- 2 Switches (2960)
- 2 PCs
- 1 Server (TFTP-enabled, Packet Tracer's built-in Server has a TFTP service)
- Serial/Copper cables as needed

---

## ⚙️ 4. Implementation Steps

### Step A — Basic Device Configuration
```
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface gigabitEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```
Repeat similarly for R2 and configure the serial link between R1 and R2.

### Step B — Routing Configuration (OSPF)
```
R1(config)# router ospf 1
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# network 10.0.0.0 0.0.0.3 area 0
```
Repeat on R2 with its own networks.

### Step C — Routing Security

**1. Enable secret & console/VTY password protection**
```
R1(config)# enable secret cisco123
R1(config)# line console 0
R1(config-line)# password console123
R1(config-line)# login
R1(config-line)# exit
R1(config)# line vty 0 4
R1(config-line)# password vty123
R1(config-line)# login
R1(config-line)# transport input ssh
```

**2. Enable SSH for remote access (instead of Telnet)**
```
R1(config)# ip domain-name mynetwork.local
R1(config)# username admin secret admin123
R1(config)# crypto key generate rsa
   (choose 1024 bits)
R1(config)# ip ssh version 2
```

**3. Encrypt all plaintext passwords**
```
R1(config)# service password-encryption
```

**4. OSPF Routing Protocol Authentication (MD5)**
```
R1(config)# interface Se0/0/0
R1(config-if)# ip ospf message-digest-key 1 md5 ospfKey123
R1(config-if)# ip ospf authentication message-digest
R1(config-if)# exit
R1(config)# router ospf 1
R1(config-router)# area 0 authentication message-digest
```
(Repeat matching config on R2 so the OSPF neighbor authentication succeeds.)

**5. Access Control List (ACL) — restrict Telnet/SSH access to management subnet only**
```
R1(config)# access-list 10 permit 192.168.10.0 0.0.0.255
R1(config)# line vty 0 4
R1(config-line)# access-class 10 in
```

**6. Disable unused services**
```
R1(config)# no ip http server
R1(config)# no cdp run
```

**7. Port Security on Access Switch**
```
Switch(config)# interface fastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 1
Switch(config-if)# switchport port-security violation shutdown
Switch(config-if)# switchport port-security mac-address sticky
```

---

### Step D — Setting Up the TFTP Server
1. Place a **Server** device in the topology, connect it to R1's Gi0/1.
2. Assign IP `192.168.10.100 /24`.
3. On the Server's **Services** tab → enable **TFTP**.
4. Verify reachability: `R1# ping 192.168.10.100`

### Step E — Backup Configuration & IOS to TFTP
```
R1# copy running-config tftp
Address or name of remote host []? 192.168.10.100
Destination filename [R1-confg]? R1-running-backup
```
```
R1# copy startup-config tftp
Address or name of remote host []? 192.168.10.100
Destination filename [R1-confg]? R1-startup-backup
```
Backing up the IOS image (optional, for full disaster recovery):
```
R1# copy flash: tftp
Source filename? c2900-universalk9-mz.SPA.bin
Address or name of remote host []? 192.168.10.100
```

### Step F — Recovery: Restoring Configuration from TFTP
Scenario: router config is lost/corrupted (e.g. after `erase startup-config` + `reload`).
```
Router> enable
Router# copy tftp running-config
Address or name of remote host []? 192.168.10.100
Source filename []? R1-running-backup
Destination filename [running-config]?
```
Then save it permanently:
```
Router# copy running-config startup-config
```

---

## ✅ 5. Verification Commands

| Purpose                          | Command                          |
|-----------------------------------|-----------------------------------|
| Check OSPF neighbors              | `show ip ospf neighbor`          |
| Check OSPF authentication status  | `show ip ospf interface`         |
| Verify SSH is active               | `show ip ssh`                    |
| Verify port security               | `show port-security interface fa0/1` |
| Verify ACL applied to VTY          | `show run \| section line vty`   |
| Confirm backup file on server      | Check TFTP server's file directory |

---

## 📷 6. Screenshots to Capture in Packet Tracer

Add these as you build the lab (place image files in an `/screenshots` folder):

1. Full network topology
2. `show ip ospf neighbor` output (proves OSPF is up)
3. `show ip ospf interface` (proves MD5 authentication enabled)
4. SSH login attempt from PC to router (proves password/SSH security)
5. Port security violation demo (unauthorized device shut down)
6. TFTP server Services tab showing TFTP = ON
7. `copy running-config tftp` command output (backup success)
8. TFTP server file list showing the backed-up config file
9. `copy tftp running-config` recovery output after simulating a config wipe
10. Final `show running-config` confirming restoration

---

## 🎯 7. Conclusion

This project shows an end-to-end workflow for securing router management access
and routing protocol exchanges, and for protecting against configuration loss
using a centralized TFTP backup/recovery process — a core skill for network
administrators and a common real-world enterprise practice.

---

## 👤 Author
**Bhagyesh**
Feel free to connect on LinkedIn / check my GitHub for more networking projects.
