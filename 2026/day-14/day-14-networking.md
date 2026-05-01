# Day 14 – Networking Fundamentals 
# PART 1 – CORE CONCEPT 

# 1. OSI vs TCP/IP 
OSI Model (7 Layers)
1. Physical     (Cable, WiFi)
2. Data Link    (MAC, Switch)
3. Network      (IP)
4. Transport    (TCP/UDP)
5. Session
6. Presentation
7. Application  (HTTP, DNS)
   
# TCP/IP Model (Practical DevOps Model)
Application   → HTTP, HTTPS, DNS
Transport     → TCP / UDP
Internet      → IP
Link          → Network hardware

# Mapping (Important for Interview)
OSI	TCP/IP
L7–L5	Application
L4	Transport
L3	Internet
L2–L1	Link

# Where Things Work
Technology	Layer
IP	Network
TCP/UDP	Transport
HTTP/HTTPS	Application
DNS	Application

# Real Example
curl https://google.com
Flow:
Application → HTTP
Transport → TCP
Internet → IP

#  PART 2 – HANDS-ON CHECKS
# Target Host
We use:
google.com

# 🔹 1. Identity (Your IP)
hostname -I
OR
ip addr show
Observation:
Shows your system IP (private/public)

# 🔹 2. Reachability
ping google.com
Observation:
latency (ms)
packet loss

Example:

0% loss = good
high latency = slow network

# 🔹 3. Path (Routing)
traceroute google.com
OR
tracepath google.com
Observation:
hops between networks
time per hop
If timeout:
firewall or routing issue

# 🔹 4. Ports
ss -tulpn
Observation:
Example:
LISTEN 0 128 *:22 sshd

 SSH running on port 22

# 🔹 5. DNS Resolution
dig google.com
OR
nslookup google.com
Observation:
domain → IP

# 🔹 6. HTTP Check
curl -I https://google.com
Observation:
HTTP/1.1 200 OK

200 = success
404 = not found
500 = server error

# 🔹 7. Connection Snapshot
netstat -an | head
Observation:
ESTABLISHED → active connections
LISTEN → waiting connections

# PART 3 – MINI TASK (PORT PROBE)
Step 1: Find port
ss -tulpn
Example:
*:22 (SSH)
Step 2: Test port
nc -zv localhost 22
Result:
Connection successful
If NOT reachable:
Check:
systemctl status ssh

#  PART 4 – REAL PRODUCTION SCENARIOS
#  Scenario 1 – Website not opening
Check:
ping domain
curl -I domain
Scenario 2 – DNS failure
dig google.com
Fix:
check /etc/resolv.conf

# Scenario 3 – Port closed
ss -tulpn
 Fix:
start service
open firewall

# Scenario 4 – Slow network
traceroute google.com
Find slow hop

#  PART 5 – INTERVIEW QUESTIONS

Q1: Difference between ping and curl?
Command	Use
ping	network connectivity
curl	application check

Q2: What is DNS?
 Converts domain → IP

Q3: What is TCP vs UDP?
TCP	UDP
Reliable	Fast
Connection-based	Connectionless

Q4: Why curl -I?
Checks HTTP status quickly

Q5: How to check open ports?
ss -tulpn
