# Day 08 – Cloud Deployment (Docker, Nginx)
# Flow 
Your Laptop → SSH → Cloud Server (EC2)
                         ↓
                    Install Nginx
                         ↓
                 Open port 80 (HTTP)
                         ↓
              Access website in browser

## 1. Commands Used

# SSH Connection
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@<IP>

# System Update
sudo apt update
sudo apt upgrade -y

# Docker Install
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

# Nginx Install
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx

# Logs
cd /var/log/nginx
cat access.log
tail -n 20 access.log

# Save logs
cp /var/log/nginx/access.log ~/nginx-logs.txt

# Download logs
scp -i your-key.pem ubuntu@<IP>:~/nginx-logs.txt .


## 2. Challenges Faced

- SSH permission denied → Fixed using chmod 400
- Nginx not opening → Fixed by opening port 80 in security group
- Logs not visible → Checked correct path /var/log/nginx

## 3. What I Learned

- How to launch EC2 instance
- How to connect using SSH
- How to install Docker and Nginx
- How to open ports using security groups
- How to read and download logs

## 4. Output Verification

- Successfully connected via SSH
- Nginx welcome page visible in browser
- Logs extracted and saved in nginx-logs.txt

## 5. Why This Matters for DevOps

- Real-world server setup
- Managing cloud infrastructure
- Deploying applications
- Debugging using logs
- Handling production-level issues
  
