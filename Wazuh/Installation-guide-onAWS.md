# Steps In SettingUp Wazuh Manager On AWS


## Step-1:

### Create an AWS EC2 Instance

- Log in to [Amazon Web Services (AWS)](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Feu-north-1.console.aws.amazon.com%2Fvpcconsole%2Fhome%3Fca-oauth-flow-id%3D3857%26hashArgs%3D%2523TrafficMirrorTargets%253A%26isauthcode%3Dtrue%26oauthStart%3D1781840069323%26region%3Deu-north-1%26state%3DhashArgsFromTB_eu-north-1_773049dfbd7e0999&client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fvpcconsole&forceMobileApp=0&code_challenge=c3M1eCgXmsWVjVGqLkJRksX5Tp9QbVgNrX0UgzvhvyY&code_challenge_method=SHA-256)
- Go to EC2 → Launch Instance
- Choose size:
  - Ubuntu Server 22.04 LTS (common choice)
  - or Amazon Linux
- Choose size:
  - Lab: t3.medium (minimun recommended)
  - Production: larger depending on endpoints
- Create/download a key paire(.pem)
- Configure Security Group rules:
  Allow:
  
| Port | Purpose |
|---|---|
| 22 | SSH |
| 1514 TCP/UDP | Wazuh agent communication |
| 1515 TCP | Agent enrollment |
| 443 | Dashboard HTTPS |
| 9200 | Indexer API (optional/admin) |

---

Restrict SSH to your IP if possible.

---

## Step-2:

### Connect To The Server

```
ssh -i wazuh-key.pem ubuntu@<EC2_PUBLIC_IP>
```
update packages:
```
sudo apt update && sudo apt upgrade -y
```

---
## Step-3:

### Install Wazuh Manager

Download the  Wazuh Manager installation script:
```
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
```
Run the installer: 
```
sudo bash wazuh-install.sh -a
```
The -a installs the complete stack:
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

---
## Step-4:

### Save The Generated Credentials

The installer outputs:
- Dashboard username
- Dashboard password
Save them.

You can also check:
```
sudo cat wazuh-passwords.txt
```

---
## Step-5:

### Verify Wazuh services 

check:
```
sudo systemctl status wazuh-manager
```
Also:
```
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```
All should show:
```
active (running)
```


---
## Step-6:

### Access the Dashboard

open:
```
https://<EC2_PUBLIC_IP>
```
Accept the certificate warning.

Login with:
```
Username: admin
Password: <generated password>
```


---
## Step-7

### Deploy Wazuh Agent

Windows-agent example:

In Wazuh Dashboard:
```
Agent → Deploy new agent
```
chose:
```
windwos
```
Enter:
- Agent name
- server ip

It generates installation commands:

Example:
```
Invoke-WebRequest -Uri https://packages.wazuh.com/... -OutFile wazuh.msi
msiexec.exe /i wazuh.msi /q WAZUH_MANAGER="YOUR_EC2_IP"
```
Start agent:
```
NET Start Wazuh
```


---
## Step-8

### Comfirm Agent Connection

On Manger:
```
sudo /var/ossec/bin/agent_control -lc
```
You should see:
```
ID Name Status
001 Windows-Agent Active
```
Or  you could just check directly in the wazuh maneger dashboard.



---
## Step-9

### Enable useful monitoring

Edit:
```
sudo nano /var/ossec/etc/ossec.conf
```
Enable things like:
- Syscheck (file integrity monitoring)
- Rootcheck
- Log collection
- Vulnerability dection

Restart:
```
sudo systemctl restart wazuh-manager
```


---
## Step-10

### Test detection

On windows agent:

Create a test file:
```
echo test > C:\test.txt
```
or trigger failed logins.

Check:
```
Wazuh Dashboard → Security-events
```
![Wazuh-Dashboard](./Image/Wazuh-Dashboard.png)
