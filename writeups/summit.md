---
title: Summit
layout: default
---

# Summit
# Summit — TryHackMe Writeup

**Platform:** TryHackMe  
**Room:** Summit  
**Difficulty:** Easy → Intermediate  
**Focus:** SOC fundamentals, detection engineering, Pyramid of Pain  

---

## Overview

The *Summit* room focuses on defensive security and detection engineering rather than exploitation.  
The goal is to analyze a simulated attack chain, identify indicators of compromise (IOCs), and apply defensive controls that progressively disrupt the attacker’s activity.

The room is structured around the **Pyramid of Pain**, starting with low-value indicators such as file hashes and moving toward high-value behavioral detections. Each task represents a different maturity level in detection capability.

---

## Environment and Scenario

The environment provides a sandbox where multiple malware samples can be executed and analyzed.  
Each sample exposes different artifacts, including:

- File hashes  
- Network connections  
- DNS queries  
- Registry modifications  
- Traffic patterns  

The defender’s role is to review the evidence produced by each sample and choose the most appropriate detection or blocking mechanism using the available tools (hash blocking, firewall rules, DNS filtering, and Sigma rules).

---

## Task 1 — Hash-Based Detection

The first malware sample is executed in the sandbox, revealing its cryptographic hashes (MD5, SHA1, SHA256).  
This represents the lowest level of the Pyramid of Pain.

**Action taken:**
- Extract the malware’s hash from the analysis report  
- Add the hash to the blocklist using the management interface  

**Result:**  
The malware is successfully blocked. However, this method is fragile, as any small modification to the file would produce a new hash and bypass the detection.

---

## Task 2 — IP-Based Blocking

The second sample establishes outbound connections to a specific external IP address.

**Action taken:**
- Identify the suspicious IP from the sandbox output  
- Create a firewall rule blocking outbound traffic to that address  

**Result:**  
The malware’s network communication is disrupted, preventing it from reaching its command infrastructure.  
While stronger than hash blocking, this method remains vulnerable to infrastructure changes.

---

## Task 3 — Domain-Based Detection

The third sample relies on DNS queries to a malicious domain rather than a static IP address.

**Action taken:**
- Identify the domain contacted by the malware  
- Block the domain at the DNS level, including subdomains  

**Result:**  
Command-and-control communication is interrupted.  
Domain blocking is more resilient than IP blocking, but can still be bypassed using techniques such as fast-flux or domain rotation.

---

## Task 4 — Registry Modification Detection

Analysis of the fourth sample shows that it modifies the Windows Registry to weaken system defenses.  
Specifically, it disables Windows Defender real-time protection.

**Action taken:**
- Create a Sigma rule detecting registry modifications related to Defender settings  
- Focus on changes to the `DisableRealtimeMonitoring` value  

**Result:**  
This detection targets attacker behavior rather than static indicators, providing significantly higher defensive value.

---

## Task 5 — Beaconing Behavior Detection

The fifth sample avoids obvious indicators and instead generates periodic outbound traffic.

**Observed behavior:**
- Network connections every 30 minutes  
- Consistent payload size of approximately 97 bytes  

**Action taken:**
- Create a Sigma rule to detect this recurring beaconing pattern  

**Result:**  
The malware is detected based purely on behavior.  
This type of detection is difficult to evade without redesigning the malware’s communication logic.

---

## Final Task — Data Collection and Exfiltration Indicators

The final logs reveal suspicious file activity in temporary directories, consistent with data staging or preparation for exfiltration.

**Action taken:**
- Create a Sigma rule detecting unexpected file creation or modification in temporary paths  
- Focus on write activity and filenames associated with the malware  

**Result:**  
The detection successfully identifies the final stage of attacker activity, completing the challenge.

---

## Key Takeaways

- The Pyramid of Pain is a practical framework for evaluating detection quality  
- Hashes and IP addresses are easy to deploy but trivial to evade  
- Behavioral detections provide the most durable defensive value  
- Effective SOC operations focus on attacker techniques rather than individual malware samples
