Cybersecurity Investigation Report: Linux Systems Security
This report documents the resolution of two security incidents investigated as part of the Jedha Cybersecurity Course. The investigation focuses on process persistence, resource exhaustion, and unauthorized network traffic.

1. Incident: High CPU Resource Exhaustion
Issue Description
A Linux workstation experienced critical performance degradation with a Load Average of 5.32. All CPU cores were pinned at 100% usage due to multiple suspicious processes.

Identification & Analysis
Using system monitoring tools, the culprit was identified as several instances of the dd command.

Command: dd if=/dev/zero of=/dev/null.

Impact: This command performs continuous null data transfers, saturating CPU cycles without any productive output.

Persistence Mechanism
The investigation revealed that the dd processes were managed by a supervisor script.

Script Location: /usr/local/bin/backup.sh.

Parent Process ID (PPID): 1 (systemd/init), indicating it was running as a system-level daemon.

Script Logic Analysis
The script used a loop to ensure the attack remained active:

Bash
#!/bin/bash

while true; do
  # Counting active dd processes
  count=$(pgrep -fc "dd if=/dev/zero of=/dev/null")
  
  # Maintaining at least 5 instances
  if [ "$count" -lt 5 ]; then
    dd if=/dev/zero of=/dev/null &
  fi
  sleep 30
done
This logic explains why terminating individual dd processes was ineffective; the script would respawn them within 30 seconds.

2. Incident: Unauthorized Network Traffic
Issue Description
Global monitoring detected suspicious outbound traffic targeting the external IP address 192.0.2.1.

Investigation Methodology
Because the traffic was intermittent, the ss (socket statistics) command was combined with watch for real-time capture.

Bash
# Monitoring outbound connections to the target IP
sudo watch -n 0.1 "ss -atp | grep 192.0.2.1"
Findings
Program Identified: curl.

Technical State: Captured in SYN-SENT state, attempting to connect to the target via port 80 (HTTP).

Process ID (PID): 18557.

Security Conclusions
Persistence: The repetitive execution of curl suggests an automated trigger, likely a cron job or a background loop.

Vulnerability: The primary issue was the lack of Egress Filtering. The system allowed internal processes to initiate arbitrary outbound connections to unknown external IPs.

Investigator Metadata
User: Pereyra.

Institution: Jedha Cybersecurity School.

System: Ubuntu Lab Jedha Environment.

Organization: Jedha Cybersecurity School

Date: February 10, 2026.
