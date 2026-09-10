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

R1(config-if)#do show ip interface breif

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

### Step D — server backup 

```
R1#copy startup-config tftp
```
Address or name of romote host[]? 10.0.0.100 {server ip same network id to router} 
Destination filename [R1-config]?
Server tftp file create name is [R1-config]
```
R1#show flash
```
[2800nm-advipserviceskg-mz151-4.m4.bin] copy to file path
```
R1#copy flash tftp
```
source filename[]? 2800nm-advipserviceskg-mz151-4.m4.bin [paste this file path]
Address or name of romote host[]? 10.0.0.100 {server ip same network id to router} 
```
```
### Step E — server Recovery
```
Router>en
Router#config t
Router(config)#interface fastethernet 0/0
Router(config)#ip address 10.0.0.10 255.0.0.0
Router(config)#no shutdown

Router#copy tftp startup-config
```
Address or name of romote host[]? 10.0.0.100 {server ip same network id to router}
source filename[]? R1-config
```
Router#show startup-config
```
All data visible 
```
Router#copy startup-config running-config
```

## 🎯 7. Conclusion

This project shows an end-to-end workflow for securing router management access
and routing protocol exchanges, and for protecting against configuration loss
using a centralized TFTP backup/recovery process — a core skill for network
administrators and a common real-world enterprise practice.

---

## 👤 Author
**Bhagyesh**
---
**linkedin.com/in/bhagyesh-dhadiwal12**
---
**github.com/bhagyeshdhadiwal04**
---
