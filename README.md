# Splunk SSH Brute Force Detection Lab

SOC home lab project demonstrating SSH brute-force detection, Splunk alerting, log analysis, incident response triage, and containment using Kali Linux, Ubuntu, and Splunk Enterprise.

---

## Overview

This project demonstrates a hands-on SOC (Security Operations Center) home lab focused on SSH brute-force detection, alerting, investigation, and incident response using Splunk Enterprise.

The lab simulates a brute-force attack from Kali Linux against an Ubuntu VM. Authentication logs from Ubuntu were forwarded to Splunk Enterprise using Splunk Universal Forwarder, where detection rules and alerts were created to identify suspicious SSH activity.

After the attack was detected, alert triage and incident response steps were performed to investigate and contain the attack.

---

## Lab Environment

### Host System
- **Operating System:** Windows 11 Home
- **SIEM Platform:** Splunk Enterprise

### Virtual Machines
- **Kali Linux VM** - Attacker machine
- **Ubuntu VM** - Victim/target machine

### Log Collection & SIEM
- **Splunk Enterprise** - Installed on Windows 11 host
- **Splunk Universal Forwarder** - Installed on Ubuntu VM

---

## Lab Architecture

```
Kali Linux (Attacker)
        ↓
  SSH Brute Force Attack
        ↓
Ubuntu VM (Victim + SSH Service)
        ↓
Splunk Universal Forwarder
        ↓
Splunk Enterprise (Windows 11)
        ↓
  Alerts & Detection
```

---

## Configuration Performed

### Ubuntu VM (Victim)
- Installed and enabled OpenSSH server
- Configured UFW firewall to allow TCP port 22 (SSH)
- Installed Splunk Universal Forwarder
- Configured `/var/log/auth.log` forwarding to Splunk Enterprise
- Created test user account for attack simulation

### Windows 11 Host (SIEM)
- Installed Splunk Enterprise
- Configured receiving port for forwarded logs (default: 9997)
- Created index for SSH logs
- Configured Windows Firewall to allow Splunk communication
- Set up detection rules and alerts

### Kali Linux VM (Attacker)
- Installed Hydra password attack tool
- Downloaded password wordlist (e.g., rockyou.txt)
- Configured network connectivity to Ubuntu VM

---

## Attack Simulation

![Hydra Attack](screenshots/hydra_attack.png)

Hydra was used from the Kali Linux VM to simulate an SSH brute-force attack against the Ubuntu VM.

### Attack Command Example

```bash
hydra -l <username> -P passwords.txt ssh://<victim-ip>
```

**Attack Methodology:**
- Password list contained multiple incorrect passwords
- One valid password included to simulate successful compromise
- Attack generated numerous failed authentication attempts
- Successful login achieved after repeated failures

---

## Detection Engineering

A Splunk detection rule and alert were created to identify SSH brute-force attacks based on failed login attempts.

### Detection Logic
- Monitor authentication logs for failed SSH login attempts
- Track source IP addresses generating failures
- Set threshold for suspicious activity (e.g., >5 failures)
- Generate alert when threshold is exceeded
- Correlate failed attempts with successful logins

### SPL Query Example

```spl
index=ssh_logs "Failed password"
| rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip, user
| where count > 5
| eval severity="high"
| table _time, src_ip, user, count, severity
```

### Alert Configuration
- **Search Window:** Last 5 minutes
- **Schedule:** Every 5 minutes
- **Threshold:** More than 5 failed attempts from single IP
- **Throttling:** Suppress duplicate alerts for 10 minutes

---

## Alert Triage & Investigation

![Alert Triage](screenshots/alert_triage.png)

After the alert triggered in Splunk, comprehensive investigation and triage were performed.

### Investigation Process

#### 1. Initial Alert Review
- Reviewed alert details and triggered search results
- Identified source IP address and target user account
- Analyzed attack timeframe and frequency

#### 2. Log Analysis
**Key Findings:**
- Multiple failed SSH login attempts observed (50+ attempts)
- All attempts originated from the same source IP address
- SSH service (TCP port 22) was the targeted vector
- Attack occurred within a compressed timeframe (seconds apart)
- Automated brute-force pattern detected
- **Successful login detected after repeated failures**

![Successful Login Detection](screenshots/successful_login_detection.png)

#### 3. Correlation Analysis
- Correlated failed authentication events with successful login
- Verified timeline of attack progression
- Confirmed no lateral movement or additional suspicious activity
- **Alert validated as True Positive**

---

## Incident Response Actions

Following the NIST Incident Response lifecycle, the following actions were taken:

### 1. Identification & Analysis
- Alert triggered and validated as malicious activity
- Confirmed brute-force attack from external IP
- Identified compromised user account
- Assessed scope: single system affected, no lateral movement

### 2. Containment
**Short-term Containment:**
- Immediately blocked attacker IP address using UFW firewall

```bash
sudo ufw deny from <attacker-ip>
sudo ufw reload
```

![UFW Containment](screenshots/ufw_containment.png)

- Verified attack activity ceased
- Monitored for additional connection attempts from other IPs

**Long-term Containment:**
- Terminated active SSH session from attacker IP
- Reviewed all active connections for suspicious activity

### 3. Eradication & Recovery
**Eradication Actions:**
- Reset compromised user account password
- Reviewed user account for unauthorized changes

**Recovery Actions:**
- Restored normal operations after containment
- Continued enhanced monitoring of SSH logs
- Verified no further unauthorized access attempts

### 4. Post-Incident Activities
**Lessons Learned:**
- Documented attack timeline and response actions
- Identified detection rule tuning needs
- Confirmed alert threshold effectiveness

**Recommendations for Future:**
- Implement SSH key-based authentication
- Configure Fail2Ban for automatic IP blocking
- Reduce SSH password authentication timeout
- Implement network segmentation
- Consider changing default SSH port (security through obscurity)

---

## Challenges Faced & Solutions

### Challenge 1: Duplicate Alert Triggering
**Problem:**
- Alert triggered repeatedly for the same incident
- Old logs were continuously re-searched
- No throttling configured, leading to alert fatigue

**Root Cause:**
- Alert search time window did not match schedule interval
- Search looked back too far in time
- Throttling settings were not configured

**Solution Applied:**
- Updated alert search time window to match schedule (5 minutes)
- Configured alert throttling to suppress duplicates for 10 minutes
- Added deduplication logic in SPL query
- Tested alert to verify single trigger per incident

### Challenge 2: Log Forwarding Delays
**Problem:**
- Initial delays in log visibility in Splunk

**Solution:**
- Verified Splunk Universal Forwarder configuration
- Checked network connectivity between Ubuntu and Splunk Enterprise
- Confirmed inputs.conf settings were correct

---

## Skills Demonstrated

### Technical Skills
- **SIEM Operations:** Splunk Enterprise administration and query development
- **Log Analysis:** Ubuntu authentication log review and correlation
- **Detection Engineering:** Creating custom detection rules and alerts
- **Incident Response:** Following IR lifecycle for containment and recovery
- **Ubantu Security:** UFW firewall configuration and SSH hardening
- **Attack Simulation:** Understanding attacker TTPs (Tactics, Techniques, Procedures)

### SOC Analyst Skills
- Alert triage and prioritization
- True Positive vs. False Positive determination
- Evidence collection and documentation
- Timeline reconstruction
- Threat intelligence correlation
- Reporting and documentation

---

## Tools & Technologies Used

| Category | Tool | Purpose |
|----------|------|---------|
| SIEM | Splunk Enterprise | Log aggregation, detection, alerting |
| Log Forwarding | Splunk Universal Forwarder | Forward Ubuntu logs to Splunk |
| Attack Platform | Kali Linux | Simulated attacker machine |
| Target System | Ubuntu Linux | Victim machine running SSH |
| Attack Tool | Hydra | SSH brute-force simulation |
| Firewall | UFW | Incident containment |
| Virtualization | VirtualBox/VMware | Lab environment hosting |

---

## Prerequisites

To replicate this lab, you will need:

### Software
- Virtualization platform (VirtualBox, VMware, or Hyper-V)
- Kali Linux ISO
- Ubuntu Server/Desktop ISO
- Splunk Enterprise (free license available)
- Splunk Universal Forwarder


### Knowledge Requirements
- Basic Linux command-line skills
- Understanding of SSH protocol
- Familiarity with SIEM concepts
- Basic networking knowledge

---

## Lab Setup Instructions

### Step 1: Install Virtual Machines
1. Create Ubuntu VM with SSH server enabled
2. Create Kali Linux VM with network tools installed
3. Ensure both VMs can communicate on the same network

### Step 2: Configure Splunk Enterprise
1. Install Splunk Enterprise on Windows 11 host
2. Configure receiving port (Settings > Forwarding and receiving)
3. Create index for SSH logs

### Step 3: Install Splunk Universal Forwarder
1. Download and install forwarder on Ubuntu VM
2. Configure inputs.conf to monitor /var/log/auth.log
3. Configure outputs.conf to forward to Splunk Enterprise
4. Restart forwarder service

### Step 4: Create Detection Rule
1. Develop SPL query for failed SSH attempts
2. Configure alert with appropriate threshold
3. Set alert schedule and throttling
4. Test alert with simulated traffic

### Step 5: Execute Attack
1. Run Hydra from Kali Linux against Ubuntu VM
2. Monitor Splunk for incoming logs
3. Verify alert triggers correctly

---

## Learning Outcomes

This project provided practical experience with:

- **SIEM Operations:** Configuring Splunk for security monitoring and alerting
- **Detection Engineering:** Creating effective detection rules with proper thresholds
- **Alert Tuning:** Reducing false positives through throttling and filtering
- **SSH Security:** Understanding brute-force attack patterns and mitigation
- **Incident Response:** Following structured IR methodology (NIST framework)
- **Log Analysis:** Investigating authentication logs for suspicious activity
- **Containment Techniques:** Using firewalls for immediate threat containment
- **Documentation:** Creating clear incident reports and technical documentation
