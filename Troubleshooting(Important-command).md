# 🪛🛠️Suricata + AWS Traffic Mirroring Troubleshooting Guide

## Overview

This document contains the troubleshooting process used while integrating Suricata IDS with AWS VPC Traffic Mirroring and Wazuh monitoring.

The main issue investigated:

> Suricata was receiving AWS network traffic but was not detecting simulated attacks against the Windows EC2 server.

The investigation focused on:
- Traffic mirroring validation
- Suricata interface capture
- Rule loading
- Alert generation
- Log analysis

---

# 1. Verify Suricata Network Interface

First confirm the network interface available on the Suricata EC2 instance.

Command:

```bash
ip a
```

Expected result:

Example:

```
ens5
172.31.x.x
```

The interface receiving mirrored traffic must match the Suricata capture interface.

---

# 2. Confirm Traffic Is Reaching Suricata

Use tcpdump to verify packets are arriving.

Command:

```bash
sudo tcpdump -i ens5
```

For RDP traffic:

```bash
sudo tcpdump -i ens5 port 3389
```

Expected:

Traffic between:

```
Kali Linux
      |
      |
Windows Server EC2 :3389
      |
      |
Suricata EC2
```

---

# 3. Check Suricata Interface Configuration

Command:

```bash
sudo suricata --dump-config | grep interface
```

Verify the configured interface.

Example:

```
af-packet.0.interface = ens5
```

---

# 4. Test Suricata Configuration

Before restarting Suricata:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

Expected:

```
Configuration provided was successfully loaded
```

---

# 5. Rule Loading Issue

Initial problem:

Suricata showed:

```
Enabled 0 rules
```

This meant rules were downloaded but not activated.

Check rule configuration:

```bash
sudo suricata --dump-config | grep rule-files
```

Expected:

```yaml
rule-files:
 - suricata.rules
```

Edit configuration:

```bash
sudo nano /etc/suricata/suricata.yaml
```

Verify:

```yaml
default-rule-path: /etc/suricata/rules

rule-files:
 - suricata.rules
```

---

# 6. Update Suricata Rules

Download/update rules:

```bash
sudo suricata-update
```

Restart Suricata:

```bash
sudo systemctl restart suricata
```

Verify:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml
```

---

# 7. Suricata Logs

## eve.json

Primary JSON alert log:

```bash
sudo tail -f /var/log/suricata/eve.json
```

Filter alerts:

```bash
sudo tail -f /var/log/suricata/eve.json | grep alert
```

Filter RDP:

```bash
sudo tail -f /var/log/suricata/eve.json | grep -i rdp
```

---

## fast.log

Check fast alerts:

```bash
sudo tail -f /var/log/suricata/fast.log
```

Note:

During testing, eve.json was producing events while fast.log was not showing expected alerts.

The main alert source used was:

```bash
/var/log/suricata/eve.json
```

---

# 8. Suricata Service Status

Check service:

```bash
sudo systemctl status suricata
```

Restart:

```bash
sudo systemctl restart suricata
```

View live logs:

```bash
sudo journalctl -u suricata -f
```

---

# 9. Verify Detection

Test traffic from Kali Linux:

Network scan:

```bash
nmap -p 3389 <Windows-Private-IP>
```

RDP brute force simulation:

```bash
hydra -l <username> -P <password-list> rdp://<Windows-IP>
```

Expected Suricata events:

```yaml
ET POLICY MS Remote Desktop Administrator Login Request

ET INFO RDP Response To External Host

ET SCAN Potential Scan
```

---

# 10. Wazuh Integration Check

Verify Suricata logs are reaching Wazuh:

```bash
sudo tail -f /var/ossec/logs/archives/archives.json | grep suricata
```

---

# Current Project Status

## Working

✅ AWS VPC Traffic Mirroring  
✅ Source ENI configured  
✅ Target ENI configured  
✅ Suricata receives mirrored packets  
✅ Suricata detects RDP-related activity  
✅ eve.json generates events  

## Remaining Improvements

⚠️ Tune detection rules for stronger brute-force identification  
⚠️ Improve Suricata alert forwarding to Wazuh  
⚠️ Add custom detection rules  
⚠️ Validate all attack scenarios

---

# Final Architecture

```yaml
Kali Linux
(Attacker)
      |
      |
      v
Windows Server EC2
(Target)
      |
      |
AWS Traffic Mirror
      |
      |
      v
Suricata IDS EC2
      |
      |
eve.json Alerts
      |
      |
      v
Wazuh SOC Dashboard
```
![Updated-Troubleshooting-image](Architecture/Image/Troubleshooting-image.jpeg)


---


## Project Status Update

Current state: ✅ Working

> The Suricata IDS deployment on AWS is now successfully receiving mirrored traffic from the Windows EC2 target through AWS    VPC Traffic Mirroring.

Previously investigated issues:
- Suricata received AWS network traffic but did not detect simulated attacks.
- Traffic mirror session was active but mirrored packets were not initially visible.
- Suricata rules were downloaded but showed `Enabled: 0 rules`.
- `fast.log` was empty while `eve.json` was receiving events.

Resolved:
- ✅ AWS Traffic Mirror source ENI verified
- ✅ AWS Traffic Mirror target ENI verified
- ✅ UDP 4789 GRE encapsulated mirror traffic verified
- ✅ Suricata interface configured correctly
- ✅ Suricata rules loaded successfully
- ✅ Custom rules enabled
- ✅ Alerts generated in eve.json
- ✅ fast.log detection working
- ✅ RDP brute-force activity detected
- ✅ Wazuh receives Suricata alerts

---

# Final Architecture

```yaml
 Kali Linux
 (Attack Simulation)
   |
   | RDP / SSH / Scan Traffic
   |
   v

 Windows Server EC2
 (Target Machine)
   |
   |
   | AWS VPC Traffic Mirroring
   | GRE over UDP 4789
   |
   v

 Suricata IDS EC2
 (Sensor)
    |
    |
    +---- eve.json
    |
    +---- fast.log
    |
    v
Wazuh Manager
(SOC Dashboard)
```

---

# Suricata Rule Loading Issue (Resolved)

## Problem

Suricata showed:

Enabled: 0 rules

and: 

SC_ERR_NO_RULES(42) No rule files match

----


## Cause

Rules existed but Suricata was searching the wrong directory.

Configuration:

```bash
 default-rule-path: /var/lib/suricata/rules
```

but custom rules existed in:

```bash
  etc/suricata/rules/
```
----

## Fix

Copy rules:

```bash
 sudo cp /etc/suricata/rules/custom.rules 
/var/lib/suricata/rules/
```

Verify:

```bash
  ls /var/lib/suricata/rules
```

Expected:

```bash
 suricata.rules
 custom.rules
```

Test:

```bash
  sudo suricata -T -c /etc/suricata/suricata.yaml
```

Expected:

```
Loaded rules
Enabled rules
```

Restart:

```bash
  sudo systemctl restart suricata
```

---

### Custom Detection Rules

Location

```bash
   sudo nano /var/lib/suricata/rules/custom.rules
```

Example RDP brute force rule:

```bash
 alert tcp any any -> $HOME_NET 3389 \
(msg:"Possible RDP Brute Force Attempt"; \
flow:to_server; \
flags:S; \
threshold:type both, track by_src, count 5, seconds 60; \
sid:1000001; rev:1;)
```

Validate:

```bash
  sudo suricata -T -c /etc/suricata/suricata.yaml
```

![Updated-Troubleshooting-image](Architecture/Image/Troubleshooting-image-completed-version.jpeg)

## Notes

⚠️This troubleshooting was performed in an isolated educational AWS lab environment.

   No real systems were targeted.

