# Day 15 – Networking Concepts 
#  PART 1 – DNS (HOW INTERNET WORKS)
#  What happens when you type google.com?

Step-by-step:

Browser asks OS → “Do we know IP for google.com?”
If not → query DNS server
DNS resolves domain → IP address
Browser connects to that IP using HTTP/HTTPS
Website loads

Simple:
Domain → DNS → IP → Server → Response

# DNS Record Types
Record	Meaning
A	Domain → IPv4
AAAA	Domain → IPv6
CNAME	Alias (domain → domain)
MX	Mail server
NS	Name server

# 🔹 Command
dig google.com
Look for:
ANSWER SECTION:
google.com.  300  IN  A  142.250.x.x

A record = IP
TTL = 300 (seconds)

#  PART 2 – IP ADDRESSING
#  What is IPv4?
 32-bit address

Example:

192.168.1.10
Structure:
Network part + Host part

#  Public vs Private IP
Public IP
Accessible on internet
Example: 8.8.8.8
Private IP
Used inside networks
Examples:
10.x.x.x
172.16–31.x.x
192.168.x.x

# 🔹 Command
ip addr show
 Look for:
inet 172.31.x.x → private IP (AWS)
PART 3 – CIDR & SUBNETTING
What is CIDR?
Example:
192.168.1.0/24
/24 = 24 bits for network

# Subnet Mask
CIDR	Mask
/24	255.255.255.0
/16	255.255.0.0
/28	255.255.255.240

#  Number of Hosts
Formula:
2^(32 - CIDR) - 2
#CIDR Table
CIDR	Subnet Mask	Total IPs	Usable Hosts
/24	255.255.255.0	256	254
/16	255.255.0.0	65536	65534
/28	255.255.255.240	16	14

# Why subnetting?
#  To:

Reduce network traffic
Improve security
Organize infrastructure
#  PART 4 – PORTS
#  What is a Port?

# A port = entry point to a service
Example:
IP = house
Port = door

#  Common Ports
Port	Service
22	SSH
80	HTTP
443	HTTPS
53	DNS
3306	MySQL
6379	Redis
27017	MongoDB

# 🔹 Command
ss -tulpn
Example:

*:22 → ssh
*:80 → nginx

# PART 5 – PUTTING IT TOGETHER

# 🔹 Q1
curl http://myapp.com:8080
Concepts involved:

DNS → resolve domain
IP → connect to server
Port 8080 → service
HTTP → application layer

# 🔹 Q2
App can't reach DB 10.0.1.50:3306

# Check:

Network connectivity (ping)
Port open (ss / telnet)
Firewall/security group
DB service running


############# Summury #####################
# Day 15 – Networking Concepts

## 1. DNS

When we type google.com:

- Browser queries DNS
- DNS resolves domain to IP
- Browser connects to server
- Website loads

### DNS Records

A → IPv4  
AAAA → IPv6  
CNAME → alias  
MX → mail server  
NS → name server  

Command:
dig google.com  

Observation:
- A record found
- TTL shows cache time  

## 2. IP Addressing

IPv4 = 32-bit address  

Example:
192.168.1.10  

### Public vs Private

Public: 8.8.8.8  
Private: 192.168.1.10  

Private ranges:
10.x.x.x  
172.16–31.x.x  
192.168.x.x  

Command:
ip addr show  

Observation:
- System IP is private  

## 3. CIDR & Subnetting

/24 = 255.255.255.0  

### Table

| CIDR | Mask | Total | Usable |
|------|------|------|--------|
| /24 | 255.255.255.0 | 256 | 254 |
| /16 | 255.255.0.0 | 65536 | 65534 |
| /28 | 255.255.255.240 | 16 | 14 |

Subnetting helps organize networks and improve performance.

## 4. Ports

Ports allow services to run on same IP  

22 → SSH  
80 → HTTP  
443 → HTTPS  
53 → DNS  
3306 → MySQL  
6379 → Redis  
27017 → MongoDB  

Command:
ss -tulpn  

Observation:
- SSH running on port 22  

## 5. Scenario Answers

curl http://myapp.com:8080:

- DNS resolves domain
- IP connects
- Port 8080 accessed
- HTTP request sent

App can't reach DB:

- Check network
- Check port
- Check firewall
- Check DB service

## What I Learned

- DNS converts domain to IP  
- CIDR defines network size  
- Ports identify services  
