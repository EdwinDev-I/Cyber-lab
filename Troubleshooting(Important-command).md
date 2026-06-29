# Suricata + AWS Traffic Mirroring Troubleshooting Guide

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
