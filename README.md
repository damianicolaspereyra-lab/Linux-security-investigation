Cybersecurity Investigation Report: Resource Exhaustion & Rogue Network Traffic
This report documents the resolution of two security incidents investigated as part of the Jedha Cybersecurity Course.

1. Incident: High CPU Resource Exhaustion
Issue Description
A Linux workstation at Skyview Media experienced critical performance degradation. Initial observation showed a Load Average of 5.32, which is extremely high for a 4-core system, with all CPUs pinned at 100.0%.

Identification & Analysis
Using htop and ps utilities, multiple instances of a resource-heavy process were identified.

Bash
# Investigative command to find the top resource consumers
# Checking CPU-intensive processes and sorting by percentage
ps aux --sort=-%cpu | head -n 10
Process Identified: dd

Exact Command: dd if=/dev/zero of=/dev/null

Impact: This command creates an infinite loop reading from /dev/zero and writing to /dev/null, consuming maximum CPU cycles with no productive output.

Persistence Mechanism
The investigation revealed that killing the dd processes was ineffective because they were immediately respawned by a supervisor script.

Parent Script: /usr/local/bin/backup.sh (PID 717).

Parent Process ID (PPID): 1 (systemd/init), indicating it was configured as a system daemon.

Script Logic: The script uses a while true loop and pgrep to ensure at least 5 instances of the dd command are running at all times.

2. Incident: Rogue Network Traffic to 192.0.2.1
Issue Description
Network monitoring detected suspicious intermittent outbound traffic from a workstation to the external IP address 192.0.2.1.

Investigation with ss
Because the traffic was ephemeral, a real-time monitoring approach was used to capture the socket information.

Bash
# Monitoring network connections to the target IP in real-time
# Using watch to catch the process during the SYN-SENT state
sudo watch -n 0.1 "ss -atp | grep 192.0.2.1"
Program Identified: curl

Technical Details: Outbound connections were captured in SYN-SENT state targeting port 80 (HTTP).

Captured PID: 18557.

Security Findings & Conclusions
Attack Persistence: The attack is persistent because the curl command is invoked repeatedly by an automated trigger (likely a cron job or a hidden loop script).

Privilege Analysis: While curl can be executed by any user to contact an external IP, the ability to maintain persistence via system directories or cron schedules often suggests an initial compromise with root or sudo privileges.

Root Cause: The primary issue allowing this activity is a lack of Egress Filtering. Without firewall rules (like iptables or ufw) to restrict outbound traffic to authorized destinations only, rogue processes can freely communicate with external servers.

Investigator Metadata
Name: Pereyra Damian

Organization: Jedha Cybersecurity School

Date: February 10, 2026.
