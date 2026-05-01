# Day 03 – Linux Commands Cheat Sheet

# The goal of Day 03 is to build strong command-line skills for:

- Process management
- File system operations
- Networking troubleshooting

# This cheat sheet is designed for quick reference during real DevOps tasks.

Why This Matters (DevOps)
In real production:
Issues are solved using commands
Logs, processes, and networks are debugged via terminal

This helps you:
Restore services faster
Reduce downtime
Become confident in troubleshooting
Process Management Commands

# ps aux
→ List all running processes

# top
→ Real-time CPU and memory usage

# htop
→ Interactive process viewer (better than top)

# kill <PID>
→ Terminate a process

# kill -9 <PID>
→ Force kill a process

# pkill <name>
→ Kill process by name

# pgrep <name>
→ Find process by name

# nice -n 10 <command>
→ Run process with priority

# renice -n 5 -p <PID>
→ Change priority of running process

# uptime
→ Show system load and uptime

# File System Commands

# ls -l
→ List files with details

# ls -a
→ Show hidden files

# cd <dir>
→ Change directory

# pwd
→ Show current directory

# mkdir <dir>
→ Create directory

# rm -rf <dir/file>
→ Remove file or directory

# cp file1 file2
→ Copy file

# mv file1 file2
→ Move or rename file

# touch file.txt
→ Create empty file

# cat file.txt
→ View file content

# less file.txt
→ View large file (scroll)

# head -n 10 file.txt
→ Show first 10 lines

# tail -n 10 file.txt
→ Show last 10 lines

# tail -f log.txt
→ Live log monitoring

# du -sh *
→ Check folder sizes

# df -h
→ Check disk usage

# find /path -name file.txt
→ Search file

# chmod 755 file.sh
→ Change file permission

# chown user:group file
→ Change ownership

# Networking Commands

ping google.com
→ Check connectivity

# ip addr
→ Show IP address

# ifconfig
→ Show network interfaces (older systems)

# netstat -tulnp
→ Show open ports

# ss -tulnp
→ Modern alternative to netstat

curl http://example.com

→ Test API or URL

# wget http://example.com/file

→ Download file

dig google.com
→ DNS lookup

nslookup google.com
→ DNS query

traceroute google.com
→ Trace network path

Logs & System Commands

# journalctl -u nginx
→ View service logs

# journalctl -xe
→ System logs

# dmesg
→ Kernel logs

# free -h
→ Memory usage

# whoami
→ Current user

# who
→ Logged-in users

# history
→ Command history

# clear
→ Clear terminal

Quick Troubleshooting Use Cases

# Check High CPU
top
ps aux --sort=-%cpu
# Check Memory Issue
free -h
ps aux --sort=-%mem
# Monitor Logs
tail -f log.txt
journalctl -u service
# Disk Full
df -h
du -sh *
# Check Network
ping google.com
curl http://example.com
ss -tulnp

# Interview Questions & Answers

Q1: How do you check running processes?
A: ps aux or top

Q2: How do you kill a process?
A: kill <PID> or kill -9 <PID>

Q3: How do you check disk usage?
A: df -h

Q4: How do you check memory usage?
A: free -h

Q5: How do you check logs?
A: journalctl or tail

Q6: How do you check open ports?
A: ss -tulnp

Q7: How do you test API?
A: curl

Q8: How do you find a file?
A: find command

Scenario-Based Questions

# Scenario 1: Server CPU is high
top
ps aux --sort=-%cpu

# Scenario 2: Application not responding
curl http://app-url
ss -tulnp

# Scenario 3: Disk is full
df -h
du -sh *

# Scenario 4: Service logs needed
journalctl -u nginx
tail -f log.txt

# Scenario 5: Network issue
ping google.com
dig google.com
traceroute google.com
