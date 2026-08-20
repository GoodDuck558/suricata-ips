# suricata-ips

An inline Intrusion Prevention System that detects network attacks in real time and automatically blocks the source using Suricata and nftables.

## Overview

Most student IDS/IPS projects stop at passive detection (alerting only). This project runs Suricata in **inline mode**, meaning it sits directly in the traffic path and can actively drop malicious connections in real time, not just log them.

Detected attacks trigger an automated response pipeline: the source IP is blocked via `nftables`, with an auto-expiry timeout (to avoid permanent lockouts from false positives) and a whitelist (to protect management access).

## Architecture

```
   Kali Linux                Ubuntu Server                 Metasploitable
   (Attacker)      ---->     (Suricata + nftables)  ---->   (Victim)
   192.168.100.10             inline / NFQUEUE mode         192.168.200.10
                               192.168.100.1 / 192.168.200.1
```

All three machines run as VirtualBox VMs on an isolated internal network, with no connection to any external or production network.

- **Attacker VM (Kali Linux):** generates attack traffic (port scans, brute-force login attempts, periodic "beacon" connections) used to test detection.
- **IPS VM (Ubuntu Server):** runs Suricata in inline/NFQUEUE mode with custom detection rules. Alerts are parsed by a response script that triggers automatic IP blocking via `nftables`.
- **Victim VM (Metasploitable 2):** intentionally vulnerable target providing realistic services to attack.

## Detection

Custom Suricata rules for:
- Pinging the ip address
- Port scanning
- SSH brute-force attempts

## Response

A Python script watches Suricata's alert output (`eve.json`), and for each valid alert:
1. Checks the source IP against a whitelist (to avoid blocking management access)
2. Checks whether it's already blocked
3. Blocks the IP via `nftables`
4. Automatically removes the block after a configured timeout
5. Logs every action for later analysis

## Evaluation (planned)

The system will be tested against both attack traffic and benign traffic to measure:
- True positive / false positive rates
- False negative rate
- Detection-to-block latency
- Impact of the block-expiry timeout on usability vs. security tradeoffs

## Repository structure

```
/suricata-rules/       Custom Suricata detection rules
/response-script/      Alert-to-block Python script + nftables logic
/docs/                 Report drafts, diagrams, notes
README.md
```

## Tech stack

- Suricata (inline/NFQUEUE mode)
- nftables
- Python
- VirtualBox (Kali Linux, Ubuntu Server, Metasploitable 2)
