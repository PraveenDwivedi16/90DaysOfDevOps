# Day 12 – Revision (Days 01–11)

## 1. Learning Plan Review

- Goal: Become DevOps engineer
- Focus: Linux + troubleshooting
- Update: Need more practice in permissions and logs

## 2. Process & Service Check
Commands:
ps aux | head  
systemctl status ssh  
journalctl -u ssh -n 5  
Observation:
- Processes running normally  
- SSH service active  
- Logs show no errors

## 3. File Practice
Commands:
echo "Revision test" >> test.txt  
chmod 600 test.txt  
ls -l test.txt  
Observation:
- File updated successfully  
- Permission changed to secure mode
  
## 4. Ownership Practice
Commands:
touch sample.txt  
chown tokyo sample.txt  
ls -l sample.txt  
Observation:
- Ownership changed correctly
- 
## 5. Top 5 Commands
- ls -l  
- ps aux  
- systemctl status  
- journalctl -u  
- chmod
- 
## 6. Self Check

Q1: 3 important commands?

systemctl status → check service  
ps aux → check processes  
journalctl → check logs  

Q2: Service health check?
systemctl status nginx  
journalctl -u nginx  

Q3: Safe permission change?
chown user:group file  
chmod 640 file  

## 7. What I Learned

- Linux basics are strong now  
- Commands are becoming faster  
- Troubleshooting approach improved  
