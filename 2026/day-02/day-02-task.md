# Day 02 – Linux Architecture, Processes, and systemd
# Objective
# The goal of Day 02 is to understand how Linux works internally, including:
# Linux architecture (Kernel, User Space, systemd)
# Process creation and management
# Process states
# systemd and service management

This knowledge is important for DevOps troubleshooting.

Why This Matters (DevOps)

Most production systems run on Linux.

This helps you:

Debug crashed services
Fix CPU and memory issues
Read logs properly
Manage services confidently
Linux Architecture

# Linux has 3 main layers:
- Hardware → Kernel → User Space
# Kernel
The kernel is the core of Linux.
# Responsibilities:
- CPU management
- Memory management
- File system handling
- Device handling
- Network operations
- Process management

Example:
When you run:
cat file.txt
The command runs in user space, but the kernel reads the file.
User Space
User space contains applications.
Examples:
bash (shell)
ls, ps, top (commands)
nginx, mysql, docker (services)
Important:
If an application crashes, the system does not crash.
systemd (init system)
After boot:
Kernel loads
systemd starts (PID 1)
systemd is responsible for:
Starting services
Managing services
Restarting failed services
Logging
Process Management

# What is a Process?
A process is a running program.
Example:
nginx
Each process has:
PID
Parent PID
CPU usage
Memory usage
State
How Processes Are Created
Linux uses:
fork() → create child process
exec() → run new program
Flow:
Shell → fork → exec → run command → exit
Process States
Running
Using CPU or ready to run
Sleeping
Waiting for input (disk, network, etc.)
Stopped
Paused manually

# Zombie
Process finished but not cleaned by parent
Idle
Waiting (mostly kernel processes)
systemd (Service Manager)
systemd manages services in Linux.
Commands:- 
# systemctl status nginx
# systemctl start nginx
# systemctl stop nginx
# systemctl restart nginx
# systemctl enable nginx
# systemctl disable nginx

Logs
# journalctl -u nginx
# journalctl -xe

Daily Linux Commands
ps aux
top
systemctl status nginx
journalctl -u nginx
df -h

# Service Not Working

systemctl status nginx
journalctl -u nginx
systemctl restart nginx

# High CPU
top
ps aux --sort=-%cpu

# High Memory

ps aux --sort=-%mem

# Disk Full
df -h
du -sh *

# Check Startup Services

systemctl list-unit-files --type=service

# Interview Questions & Answers

Q1: What is Linux Kernel?
A: Core of OS that manages hardware and resources.

Q2: What is systemd?
A: Service manager that controls system services.

Q3: What is PID 1?
A: First process started (systemd).

Q4: Difference between ps and top?
A: ps = snapshot, top = live view.

Q5: What is Zombie process?
A: Completed process not cleaned by parent.

Q6: How to check logs?
journalctl -u nginx

Q7: How to restart service?
systemctl restart nginx

Q8: What happens when you run command?
Shell → fork → exec → run → exit

# Scenario-Based Questions

# Scenario 1: Nginx is down

systemctl status nginx
journalctl -u nginx
systemctl restart nginx

# Scenario 2: CPU is high

top
ps aux --sort=-%cpu

# Scenario 3: Service not starting after reboot

systemctl status app
systemctl is-enabled app
Enable if needed:
systemctl enable app

# Scenario 4: Disk full
df -h
du -sh *

# Scenario 5: Zombie processes
Cause: Parent not cleaning child
Fix: Restart parent or fix application

# Quick Revision Notes

Kernel = core
User space = apps
systemd = service manager
fork + exec = process creation
Zombie = finished but not cleaned
systemctl = manage services
journalctl = logs
