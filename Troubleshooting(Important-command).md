# Suricata + AWS Traffic Mirroring Troubleshooting Guide

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

```
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

```
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
[TroubleShooting-Image](Architecture/Image/Troubleshooting-image.jpeg)

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

```
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

---

## Notes

This troubleshooting was performed in an isolated educational AWS lab environment.

No real systems were targeted.
