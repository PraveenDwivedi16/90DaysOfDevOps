# Day 05 – Linux Troubleshooting Runbook (CPU, Memory, Logs)
- Objective

# The goal of this drill is to practice a structured troubleshooting approach by:
Capturing system health (CPU, Memory, Disk, Network)
Inspecting logs
Building a repeatable runbook
Target Service

Service: ssh
Process: sshd

# 1. Environment Basics
# Command 1
uname -a
Output:
Linux ubuntu 5.15.0-78-generic x86_64 GNU/Linux
Observation:
System is running Linux kernel 5.15 on Ubuntu.

# Command 2
cat /etc/os-release
Output:
NAME="Ubuntu"
VERSION="22.04 LTS"
Observation:
OS is Ubuntu 22.04 LTS.

# 2. Filesystem Sanity
# Command 3
mkdir /tmp/runbook-demo

# Command 4
cp /etc/hosts /tmp/runbook-demo/hosts-copy && ls -l /tmp/runbook-demo
Output:
-rw-r--r-- 1 root root 250 hosts-copy
Observation:
Filesystem is working correctly (read/write operations successful).

# 3. CPU & Memory Snapshot
# Command 5
top
Output (sample):
%Cpu(s): 5.0 us, 2.0 sy, 93.0 id
MiB Mem : 2000 total, 1200 free
Observation:
CPU usage is low, system is mostly idle. Memory usage is normal.

# Command 6
free -h
Output:
total used free
2.0Gi 500Mi 1.2Gi
Observation:
Enough free memory available, no memory pressure.

# 4. Disk & IO Snapshot
# Command 7
df -h
Output:
/dev/sda1 20G 10G 10G 50%
Observation:
Disk usage is at 50%, no immediate issue.

# Command 8
du -sh /var/log
Output:
200M /var/log
Observation:
Log directory size is moderate, not excessive.

# 5. Network Snapshot
# Command 9
ss -tulnp | grep ssh
Output:
LISTEN 0 128 *:22 : users:(("sshd",pid=1025))
Observation:
SSH service is listening on port 22.

# Command 10
curl -I http://localhost
Output:
HTTP/1.1 200 OK
Observation:
Local network/service responding correctly.

# 6. Logs Reviewed
# Command 11
journalctl -u ssh -n 10
Output:
sshd[1025]: Accepted password for user
sshd[1025]: Session closed
Observation:
No recent errors, normal login activity.

# Command 12
tail -n 10 /var/log/syslog
Output:
sshd[1025]: login successful
sshd[1025]: session closed
Observation:
Logs confirm successful SSH connections.

# 7. Quick Findings
CPU usage is low → no performance issue
Memory is sufficient → no pressure
Disk usage normal → no storage issue
Network port (22) is active → service reachable
Logs show no errors → service healthy

# 8. Mini Runbook (Steps Followed)
Checked OS and kernel version
Verified filesystem read/write
Checked CPU and memory usage
Verified disk usage
Checked network port
Reviewed logs for service

# 9. If This Worsens (Next Steps)
Restart service
systemctl restart ssh
Increase log visibility
journalctl -u ssh -f
Deep debugging
strace -p <PID>
Check resource spikes
top / htop
Check firewall rules
iptables -L
Analyze CPU, memory, disk, and network
Read and interpret logs
Follow a structured troubleshooting approach
