
# Step to Configure Wazuh to read Suricata logs file:

## Step1: verify Suricata is reading logs first
```bash
 sudo tail -f /var/log/suricata/eve.json
```
## Step 2: install Wazuh-agent & check the status 
```bash
 curl -sO https://packages.wazuh.com/4.x/apt/wazuh-agent.deb
sudo systemctl status wazuh-agent
```

## Step 3: configure Wazuh to read Suricata logs
```bash
 sudo nano /var/ossec/etc/ossec.conf
```
### In the <ossec_config> section add:
```bash
 <localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```
## Step 4: restart Wazuh and check status 
```bash
 sudo systemctl restart wazuh-agent

sudo systemctl status wazuh-agent
```
---
## Image Example:

![image](./Image/Clear-incident.png)
