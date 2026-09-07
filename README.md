# Routing Security & Backup–Recovery using TFTP Server (Cisco Packet Tracer)

## 📌 1. Project Objective

This project demonstrates how to secure router-to-router communication using
routing security best practices, and how to back up and recover Cisco device
configurations (running-config, startup-config, IOS image) using a **TFTP
server**. It simulates a small enterprise network

---

## 🛠 3. Tools & Requirements

- Cisco Packet Tracer (9.x)
- 2 Routers (Type: 2811 )
- 2 Switches (295DT)
- 8 PCs
- 1 Server (TFTP-enabled, Packet Tracer's built-in Server has a TFTP service)
- Serial/Copper cables as needed

---

## 🚀 4. Implementation Steps

### Step A - Router password security / Type 5 Encription 
```
Router> enable
Router# configure terminal
Router(config)# username admin secret cisco@123
Router(config)# line console 0
Router(config-line)# login local
Router(config-line)# exit

Router(config)# line aux 0
Router(config-line)# login local
Router(config-line)# exit

Router(config)#enable secret cisco123

```
### Step B - Router configuration / IP addressing 
```
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface fastethernet 0/0
R1(config-if)# ip address 10.0.1.1 255.0.0.0
R1(config-if)# no shutdown
R1(config-if)# exit

R2(config-if)#do show ip interface breif

Router> enable
Router# configure terminal
Router(config)# hostname R2
R2(config)# interface fastethernet 0/1
R2(config-if)# ip address 192.168.1.1 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit

R2(config-if)#do show ip interface breif

```
### Step C — Data copy RAM to NVRAM 
```
R1#copy running-config startup-config
R1#show startup-config

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
Feel free to connect on LinkedIn / check my 
GitHub for more networking projects.
