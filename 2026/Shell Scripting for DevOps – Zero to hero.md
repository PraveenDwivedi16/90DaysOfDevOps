Shell Scripting for DevOps – Zero to Hero Guide
0. Shell Scripting DevOps me Kyu Important Hai?
DevOps engineer ka daily kaam hota hai:
Server manage karnaLogs check karnaBackup lenaDeployment automate karnaService restart/check karnaMonitoring karnaCron jobs setup karnaCloud servers configure karnaCI/CD pipeline commands likhnaDocker/Kubernetes commands automate karna
In sab me shell scripting ka use hota hai.
Example:
Aap manually ye kar sakte ho:
cd /var/www/appgit pullnpm installnpm run buildsudo systemctl restart app
Lekin baar-baar manually karna risky hai.
Isko script bana sakte ho:
#!/bin/bashcd /var/www/appgit pullnpm installnpm run buildsudo systemctl restart appecho "Deployment completed"
Yehi DevOps automation ka base hai.

1. Shell Kya Hai?
Shell ek program hai jo user ke commands ko operating system tak bhejta hai.
Example:
lspwdcd /tmpmkdir test
Aap command type karte ho, shell usko OS ko samjhata hai.
Common shells:
shbashzshfish
DevOps me sabse common:
bash

2. Shell Script Kya Hai?
Shell script ek file hoti hai jisme multiple Linux commands likhe hote hain.
Example:
#!/bin/bashecho "Server update started"sudo apt updatesudo apt upgrade -yecho "Server update completed"
File ka naam:
update_server.sh
Run:
chmod +x update_server.sh./update_server.sh

3. Basic Script Structure
#!/bin/bash# This is a commentecho "Hello DevOps"
Explanation
Shebang
#!/bin/bash
Ye batata hai ki script ko Bash shell se run karna hai.
Comment
# This is a comment
Comments explanation ke liye hote hain. Shell ignore karta hai.
echo
echo "Hello DevOps"
Terminal par message print karta hai.

4. Script Run Kaise Karte Hain?
Create file:
nano hello.sh
Code:
#!/bin/bashecho "Hello DevOps"
Permission do:
chmod +x hello.sh
Run:
./hello.sh
Alternative:
bash hello.sh
Difference
./hello.sh
Shebang use karta hai.
bash hello.sh
Direct Bash se run karta hai.

5. Variables
Variable value store karta hai.
NAME="Hardik"ROLE="DevOps Engineer"echo "Hello, I am $NAME and I am a $ROLE"
Output:
Hello, I am Hardik and I am a DevOps Engineer
Important Rule
Wrong:
NAME = "Hardik"
Correct:
NAME="Hardik"
Shell scripting me = ke around space nahi hota.

6. Quotes
Double Quotes
Variable expand hota hai.
NAME="Hardik"echo "Hello $NAME"
Output:
Hello Hardik
Single Quotes
Variable expand nahi hota.
NAME="Hardik"echo 'Hello $NAME'
Output:
Hello $NAME
Best Practice
Always variables ko double quotes me use karo:
echo "$NAME"
Especially file paths me:
rm "$FILE_NAME"

7. User Input with read
#!/bin/bashread -p "Enter your name: " NAMEecho "Hello $NAME"
Run:
./input.sh
Output:
Enter your name: HardikHello Hardik

8. Command-Line Arguments
Script run karte time values pass karna.
./greet.sh Hardik DevOps
Inside script:
#!/bin/bashecho "Script name: $0"echo "First argument: $1"echo "Second argument: $2"echo "Total arguments: $#"echo "All arguments: $@"
Output:
Script name: ./greet.shFirst argument: HardikSecond argument: DevOpsTotal arguments: 2All arguments: Hardik DevOps

9. Exit Codes
Har command success ya failure status return karta hai.
echo $?
Common:
0 = success1 = failure2 = command misuse
Example:
ls /tmpecho $?
Agar command successful hai:
0
Example failure:
ls /wrong-pathecho $?
Output:
1 or 2

10. If-Else Conditions
Basic Syntax
if [ condition ]; then    commandelse    commandfi
Example:
#!/bin/bashread -p "Enter number: " NUMBERif [ "$NUMBER" -gt 0 ]; then    echo "Positive"elif [ "$NUMBER" -lt 0 ]; then    echo "Negative"else    echo "Zero"fi

11. Integer Operators
OperatorMeaning-eqequal-nenot equal-gtgreater than-ltless than-gegreater than or equal-leless than or equal
Example:
AGE=20if [ "$AGE" -ge 18 ]; then    echo "Adult"else    echo "Minor"fi

12. String Operators
OperatorMeaning=equal!=not equal-zempty-nnot empty
Example:
ENV="prod"if [ "$ENV" = "prod" ]; then    echo "Production environment"fi
Empty check:
NAME=""if [ -z "$NAME" ]; then    echo "Name is empty"fi

13. File Test Operators
OperatorMeaning-fregular file exists-ddirectory exists-efile/directory exists-rreadable-wwritable-xexecutable-sfile exists and not empty
Example:
FILE="app.log"if [ -f "$FILE" ]; then    echo "File exists"else    echo "File not found"fi
Directory check:
DIR="/var/log"if [ -d "$DIR" ]; then    echo "Directory exists"fi

14. Logical Operators
AND
mkdir test && cd test
Second command tab chalega jab first successful hoga.
OR
mkdir test || echo "Directory creation failed"
Second command tab chalega jab first fail hoga.
NOT
if [ ! -f "app.log" ]; then    echo "File missing"fi

15. Case Statement
Multiple choices ke liye case clean hota hai.
#!/bin/bashread -p "Enter environment dev/staging/prod: " ENVcase "$ENV" in    dev)        echo "Development selected"        ;;    staging)        echo "Staging selected"        ;;    prod)        echo "Production selected"        ;;    *)        echo "Invalid environment"        ;;esac

16. Loops
for Loop
List ke through loop.
for fruit in apple banana mangodo    echo "$fruit"done
Range Loop
for number in {1..5}do    echo "$number"done
C-style Loop
for (( i=1; i<=5; i++ ))do    echo "$i"done

17. while Loop
Condition true rahe tab tak loop chalta hai.
COUNT=5while [ "$COUNT" -gt 0 ]do    echo "$COUNT"    COUNT=$((COUNT - 1))done

18. until Loop
Condition true hone tak loop chalta hai.
COUNT=1until [ "$COUNT" -gt 5 ]do    echo "$COUNT"    COUNT=$((COUNT + 1))done

19. break and continue
break
Loop stop karta hai.
for i in 1 2 3 4 5do    if [ "$i" -eq 3 ]; then        break    fi    echo "$i"done
Output:
12
continue
Current iteration skip karta hai.
for i in 1 2 3 4 5do    if [ "$i" -eq 3 ]; then        continue    fi    echo "$i"done
Output:
1245

20. Functions
Function reusable block hota hai.
greet() {    echo "Hello DevOps"}greet
Function with Arguments
greet() {    echo "Hello $1"}greet "Hardik"
Output:
Hello Hardik
Function with Local Variable
add() {    local A=$1    local B=$2    local SUM=$((A + B))    echo "$SUM"}RESULT=$(add 10 20)echo "Result is $RESULT"
Output:
Result is 30

21. Return vs Echo in Functions
return
return exit status ke liye use hota hai.
check_file() {    if [ -f "$1" ]; then        return 0    else        return 1    fi}check_file "app.log"if [ $? -eq 0 ]; then    echo "File exists"else    echo "File missing"fi
echo
Actual value return karne ke liye echo use hota hai.
get_date() {    echo "$(date +%Y-%m-%d)"}TODAY=$(get_date)echo "$TODAY"

22. Strict Mode
DevOps scripts me ye important hai.
#!/bin/bashset -euo pipefail
Meaning
FlagMeaningset -ecommand fail ho to script stopset -uundefined variable use ho to errorset -o pipefailpipeline me koi command fail ho to pipeline fail
Example:
#!/bin/bashset -euo pipefailecho "Starting"wrongcommandecho "This will not run"

23. Debugging
set -x
Command execution trace show karta hai.
#!/bin/bashset -xNAME="DevOps"echo "$NAME"
Run without editing:
bash -x script.sh

24. trap
Script exit hone par cleanup karne ke liye.
#!/bin/bashset -euo pipefailcleanup() {    echo "Cleaning temporary files"    rm -f /tmp/my-temp-file}trap cleanup EXITtouch /tmp/my-temp-fileecho "Doing work..."
Use cases:
Temporary files delete karnaLock files remove karnaFailure cleanup

25. Useful Linux Commands for DevOps Scripting
grep
Search text.
grep "ERROR" app.loggrep -i "error" app.loggrep -n "CRITICAL" app.loggrep -c "ERROR" app.loggrep -E "ERROR|Failed" app.log
awk
Columns process karna.
awk '{print $1}' file.txtawk -F: '{print $1}' /etc/passwdawk '/ERROR/ {print}' app.log
sed
Text replace/edit.
sed 's/old/new/g' file.txtsed -i 's/foo/bar/g' config.txtsed '/DEBUG/d' app.log
cut
Column extract.
cut -d',' -f1 data.csvcut -d':' -f1 /etc/passwd
sort and uniq
sort file.txtsort file.txt | uniqsort file.txt | uniq -csort file.txt | uniq -c | sort -rn
find
find /var/log -name "*.log"find /var/log -name "*.log" -mtime +7find /var/log -name "*.log" -mtime +7 -delete
tar
Backup archive create karna.
tar -czf backup.tar.gz /var/www/html
df and free
Disk and memory.
df -hfree -h
ps
Processes.
ps auxps -eo pid,comm,%cpu,%mem --sort=-%cpu | head

26. Cron Jobs
Cron scheduled tasks ke liye hota hai.
View:
crontab -l
Edit:
crontab -e
Syntax:
* * * * * command│ │ │ │ ││ │ │ │ └── Day of week│ │ │ └──── Month│ │ └────── Day of month│ └──────── Hour└────────── Minute
Examples:
Every day 2 AM:
0 2 * * * /path/script.sh
Every 5 minutes:
*/5 * * * * /path/health_check.sh
Every Sunday 3 AM:
0 3 * * 0 /path/backup.sh
Always use absolute path in cron.

27. DevOps Real-World Script Examples
Example 1: Service Check
#!/bin/bashset -euo pipefailSERVICE="nginx"if systemctl is-active --quiet "$SERVICE"; then    echo "$SERVICE is running"else    echo "$SERVICE is not running"fi
Example 2: Disk Alert
#!/bin/bashset -euo pipefailTHRESHOLD=80USAGE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')if [ "$USAGE" -gt "$THRESHOLD" ]; then    echo "Disk usage is high: $USAGE%"else    echo "Disk usage is normal: $USAGE%"fi
Example 3: Backup Script
#!/bin/bashset -euo pipefailSOURCE="/var/www/html"DEST="/backup"DATE=$(date +%Y-%m-%d)mkdir -p "$DEST"tar -czf "$DEST/backup-$DATE.tar.gz" "$SOURCE"echo "Backup completed: backup-$DATE.tar.gz"
Example 4: Log Analyzer
#!/bin/bashset -euo pipefailLOG_FILE=$1if [ ! -f "$LOG_FILE" ]; then    echo "Log file not found"    exit 1fiecho "Total lines:"wc -l < "$LOG_FILE"echo "Error count:"grep -Ei "ERROR|Failed" "$LOG_FILE" | wc -lecho "Critical events:"grep -n "CRITICAL" "$LOG_FILE" || true
Example 5: Deployment Script
#!/bin/bashset -euo pipefailAPP_DIR="/var/www/myapp"SERVICE="myapp"cd "$APP_DIR"git pullnpm installnpm run buildsudo systemctl restart "$SERVICE"echo "Deployment completed"

28. Senior-Level Best Practices
1. Always Use Shebang
#!/bin/bash
2. Use Strict Mode
set -euo pipefail
3. Quote Variables
Good:
rm "$FILE"
Bad:
rm $FILE
4. Validate Arguments
if [ $# -ne 1 ]; then    echo "Usage: ./script.sh <file>"    exit 1fi
5. Check Files/Directories Before Use
if [ ! -d "$DIR" ]; then    echo "Directory missing"    exit 1fi
6. Use Functions
Bad:
echo "Start"# many commandsecho "End"
Good:
main() {    check_disk    check_memory}main
7. Use Meaningful Names
Good:
BACKUP_DIR="/backup"LOG_FILE="/var/log/app.log"
Bad:
x="/backup"y="/var/log/app.log"
8. Log Important Output
echo "$(date): Backup completed" >> backup.log
9. Avoid Hardcoding When Possible
Use arguments or variables.
SOURCE_DIR=$1BACKUP_DIR=$2
10. Test Before Cron
First run manually:
./backup.sh
Then add to cron.

29. Common Mistakes
Mistake 1: Space Around Equal
Wrong:
NAME = "Hardik"
Correct:
NAME="Hardik"
Mistake 2: Missing Spaces in If
Wrong:
if ["$A" -eq 1]; then
Correct:
if [ "$A" -eq 1 ]; then
Mistake 3: Not Quoting Variables
Wrong:
if [ -f $FILE ]; then
Correct:
if [ -f "$FILE" ]; then
Mistake 4: Forgetting fi
Every if ends with:
fi
Mistake 5: Cron Relative Path
Wrong:
0 2 * * * ./backup.sh
Correct:
0 2 * * * /home/ubuntu/scripts/backup.sh

30. Interview Questions and Answers
Q1. What is shell scripting?
Shell scripting means writing Linux commands inside a file to automate tasks.
Example:
#!/bin/bashecho "Hello"

Q2. Why is shell scripting important in DevOps?
It is used for automation, deployment, backup, monitoring, log analysis, server setup, and cron jobs.

Q3. What is shebang?
Shebang tells the system which interpreter should run the script.
#!/bin/bash

Q4. Difference between ./script.sh and bash script.sh?
./script.sh uses the shebang and requires execute permission.
bash script.sh runs the file directly using Bash and does not require execute permission.

Q5. How do you make a script executable?
chmod +x script.sh

Q6. How do you declare a variable?
NAME="DevOps"
No spaces around =.

Q7. Difference between single quotes and double quotes?
Double quotes expand variables.
echo "$NAME"
Single quotes print literal text.
echo '$NAME'

Q8. What is $1?
$1 is the first command-line argument.
Example:
./script.sh Hardik
Inside script:
echo "$1"
Output:
Hardik

Q9. What is $#?
$# gives the total number of arguments.

Q10. What is $@?
$@ gives all arguments passed to the script.

Q11. What is $??
$? stores the exit code of the last executed command.

Q12. Difference between -eq and =?
-eq is for integer comparison.
[ "$A" -eq "$B" ]
= is for string comparison.
[ "$NAME" = "DevOps" ]

Q13. What does -f do?
-f checks whether a regular file exists.
[ -f app.log ]

Q14. What does -d do?
-d checks whether a directory exists.
[ -d /var/log ]

Q15. What is a for loop?
A for loop repeats commands over a list.
for i in 1 2 3do    echo "$i"done

Q16. What is a while loop?
A while loop runs while condition is true.
while [ "$COUNT" -gt 0 ]do    echo "$COUNT"    COUNT=$((COUNT - 1))done

Q17. What is a function in shell script?
A function is reusable code block.
greet() {    echo "Hello"}greet

Q18. What is local?
local makes a variable available only inside a function.
demo() {    local NAME="DevOps"}

Q19. What is set -e?
It exits the script when any command fails.

Q20. What is set -u?
It treats undefined variables as errors.

Q21. What is set -o pipefail?
It makes pipeline fail if any command inside pipeline fails.

Q22. What is set -x?
It enables debug mode and shows commands before execution.

Q23. What is trap?
trap runs a command/function when script exits or receives a signal.
Example:
trap cleanup EXIT

Q24. How do you check if a service is running?
systemctl is-active --quiet nginx

Q25. How do you schedule a script using cron?
Edit cron:
crontab -e
Example every day 2 AM:
0 2 * * * /path/backup.sh

Q26. How do you count ERROR lines in a log?
grep -c "ERROR" app.log
Or multiple patterns:
grep -Ei "ERROR|Failed" app.log | wc -l

Q27. How do you find files older than 7 days?
find /path -type f -mtime +7

Q28. How do you delete files older than 7 days?
find /path -type f -mtime +7 -delete

Q29. How do you create a backup using tar?
tar -czf backup.tar.gz /source/path

Q30. What are shell scripting best practices?
Use shebang, strict mode, quote variables, validate inputs, use functions, log output, handle errors, and test before scheduling in cron.

31. 25 DevOps Shell Scripting Tasks: Basic to Advanced
Task 1: Hello DevOps Script
What to do
Create a script that prints:
Hello DevOps
Why
To understand basic script creation and execution.
How
nano hello_devops.sh
Code:
#!/bin/bashecho "Hello DevOps"
Run:
chmod +x hello_devops.sh./hello_devops.sh

Task 2: User Input Script
What to do
Ask user name and print greeting.
Why
DevOps scripts often need input from users.
How
#!/bin/bashread -p "Enter your name: " NAMEecho "Hello $NAME"

Task 3: Argument Validation Script
What to do
Script should accept one filename as argument.
Why
Many automation scripts work with files.
How
#!/bin/bashif [ $# -ne 1 ]; then    echo "Usage: ./check_file.sh <filename>"    exit 1fiFILE=$1echo "File provided: $FILE"

Task 4: File Existence Checker
What to do
Check if a file exists.
Why
Before reading logs/configs, file check is important.
How
#!/bin/bashFILE=$1if [ -f "$FILE" ]; then    echo "File exists"else    echo "File not found"fi

Task 5: Directory Checker
What to do
Check if directory exists. If not, create it.
Why
Useful for backups and logs.
How
#!/bin/bashDIR=$1if [ ! -d "$DIR" ]; then    mkdir -p "$DIR"    echo "Directory created: $DIR"else    echo "Directory already exists"fi

Task 6: Number Checker
What to do
Check whether number is positive, negative, or zero.
Why
To learn numeric conditions.
How
#!/bin/bashread -p "Enter number: " NUMBERif [ "$NUMBER" -gt 0 ]; then    echo "Positive"elif [ "$NUMBER" -lt 0 ]; then    echo "Negative"else    echo "Zero"fi

Task 7: Service Status Checker
What to do
Check if nginx is running.
Why
Service monitoring is daily DevOps work.
How
#!/bin/bashSERVICE="nginx"if systemctl is-active --quiet "$SERVICE"; then    echo "$SERVICE is running"else    echo "$SERVICE is stopped"fi

Task 8: Disk Usage Alert
What to do
Alert if disk usage is above 80%.
Why
Disk full issues are common production problems.
How
#!/bin/bashTHRESHOLD=80USAGE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')if [ "$USAGE" -gt "$THRESHOLD" ]; then    echo "Alert: Disk usage is $USAGE%"else    echo "Disk usage is normal: $USAGE%"fi

Task 9: Memory Usage Reporter
What to do
Print memory usage.
Why
Memory check helps server monitoring.
How
#!/bin/bashfree -h
Advanced:
#!/bin/bashUSED=$(free -m | awk 'NR==2 {print $3}')TOTAL=$(free -m | awk 'NR==2 {print $2}')echo "Memory Used: ${USED}MB / ${TOTAL}MB"

Task 10: Top CPU Process Script
What to do
Show top 5 CPU processes.
Why
Useful in troubleshooting high CPU issues.
How
#!/bin/bashps -eo pid,comm,%cpu,%mem --sort=-%cpu | head -n 6

Task 11: Log Error Counter
What to do
Count ERROR and Failed lines in a log file.
Why
Log monitoring is key DevOps skill.
How
#!/bin/bashLOG_FILE=$1if [ ! -f "$LOG_FILE" ]; then    echo "Log file not found"    exit 1figrep -Ei "ERROR|Failed" "$LOG_FILE" | wc -l

Task 12: Critical Event Finder
What to do
Find CRITICAL lines with line numbers.
Why
Production issue detection.
How
#!/bin/bashLOG_FILE=$1grep -n "CRITICAL" "$LOG_FILE" || echo "No critical events found"

Task 13: Backup Script
What to do
Backup source directory to destination.
Why
Backup automation is common DevOps responsibility.
How
#!/bin/bashset -euo pipefailSOURCE=$1DEST=$2DATE=$(date +%Y-%m-%d)if [ ! -d "$SOURCE" ]; then    echo "Source directory missing"    exit 1fimkdir -p "$DEST"tar -czf "$DEST/backup-$DATE.tar.gz" "$SOURCE"echo "Backup completed"

Task 14: Delete Old Backups
What to do
Delete backup files older than 14 days.
Why
Avoid disk full due to old backups.
How
#!/bin/bashBACKUP_DIR=$1find "$BACKUP_DIR" -type f -name "*.tar.gz" -mtime +14 -deleteecho "Old backups deleted"

Task 15: Log Rotation Script
What to do
Compress .log files older than 7 days.
Why
Log files can grow large.
How
#!/bin/bashLOG_DIR=$1find "$LOG_DIR" -type f -name "*.log" -mtime +7 -exec gzip {} \;echo "Old logs compressed"

Task 16: Package Installer
What to do
Install nginx, curl, wget if missing.
Why
Server bootstrap automation.
How
#!/bin/bashif [ "$EUID" -ne 0 ]; then    echo "Run as root"    exit 1fiPACKAGES="nginx curl wget"for package in $PACKAGESdo    if dpkg -s "$package" &> /dev/null; then        echo "$package already installed"    else        apt install "$package" -y    fidone

Task 17: Deployment Script
What to do
Pull latest code, install dependencies, restart service.
Why
Deployment automation.
How
#!/bin/bashset -euo pipefailAPP_DIR="/var/www/myapp"SERVICE="myapp"cd "$APP_DIR"git pullnpm installnpm run buildsudo systemctl restart "$SERVICE"echo "Deployment completed"

Task 18: Health Check Script
What to do
Check website URL status.
Why
Used in monitoring.
How
#!/bin/bashURL="https://example.com"STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$URL")if [ "$STATUS" -eq 200 ]; then    echo "Website is healthy"else    echo "Website issue. Status: $STATUS"fi

Task 19: Docker Container Status Script
What to do
Check running Docker containers.
Why
Docker is common in DevOps.
How
#!/bin/bashdocker ps
Advanced:
#!/bin/bashCONTAINER="myapp"if docker ps --format '{{.Names}}' | grep -q "^$CONTAINER$"; then    echo "$CONTAINER is running"else    echo "$CONTAINER is not running"fi

Task 20: Docker Cleanup Script
What to do
Remove stopped containers and unused images.
Why
Free disk space.
How
#!/bin/bashdocker container prune -fdocker image prune -fecho "Docker cleanup completed"

Task 21: Environment-Based Script
What to do
Accept dev/staging/prod and print deployment target.
Why
Used in CI/CD pipelines.
How
#!/bin/bashENV=$1case "$ENV" in    dev)        echo "Deploying to development"        ;;    staging)        echo "Deploying to staging"        ;;    prod)        echo "Deploying to production"        ;;    *)        echo "Usage: ./deploy.sh dev|staging|prod"        exit 1        ;;esac

Task 22: CSV Parser
What to do
Read users from CSV.
Why
Automation with data files.
How
CSV:
name,roleHardik,DevOpsAmit,Developer
Script:
#!/bin/bashwhile IFS=',' read -r NAME ROLEdo    echo "Name: $NAME, Role: $ROLE"done < users.csv
Skip header:
tail -n +2 users.csv | while IFS=',' read -r NAME ROLEdo    echo "Name: $NAME, Role: $ROLE"done

Task 23: Log Report Generator
What to do
Generate report with total lines, errors, critical events.
Why
Real-world log analysis.
How
#!/bin/bashset -euo pipefailLOG_FILE=$1REPORT="report-$(date +%Y-%m-%d).txt"{    echo "Log Report"    echo "Date: $(date)"    echo "Total lines: $(wc -l < "$LOG_FILE")"    echo "Errors: $(grep -Ei "ERROR|Failed" "$LOG_FILE" | wc -l)"    echo "Critical events:"    grep -n "CRITICAL" "$LOG_FILE" || true} > "$REPORT"echo "Report generated: $REPORT"

Task 24: Cron-Based Backup
What to do
Schedule backup script daily at 2 AM.
Why
Automation should run without manual effort.
How
Edit cron:
crontab -e
Add:
0 2 * * * /home/ubuntu/scripts/backup.sh /var/www/html /backup >> /var/log/backup.log 2>&1

Task 25: Complete Server Maintenance Script
What to do
Create one script that:
Checks diskChecks memoryChecks serviceCompresses old logsCreates backupWrites logs
Why
This is real-world DevOps automation.
How
#!/bin/bashset -euo pipefailLOG_FILE="/tmp/maintenance.log"SERVICE="nginx"LOG_DIR="/tmp/myapp-logs"SOURCE_DIR="/tmp/source-app"BACKUP_DIR="/tmp/backups"log() {    echo "$(date '+%Y-%m-%d %H:%M:%S') : $1" | tee -a "$LOG_FILE"}check_disk() {    USAGE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')    log "Disk usage: $USAGE%"}check_memory() {    MEMORY=$(free -h | awk 'NR==2 {print $3 "/" $2}')    log "Memory usage: $MEMORY"}check_service() {    if systemctl is-active --quiet "$SERVICE"; then        log "$SERVICE is running"    else        log "$SERVICE is stopped"    fi}rotate_logs() {    if [ -d "$LOG_DIR" ]; then        find "$LOG_DIR" -type f -name "*.log" -mtime +7 -exec gzip {} \;        log "Old logs compressed"    else        log "Log directory not found: $LOG_DIR"    fi}backup_app() {    DATE=$(date +%Y-%m-%d)    mkdir -p "$BACKUP_DIR"    if [ -d "$SOURCE_DIR" ]; then        tar -czf "$BACKUP_DIR/backup-$DATE.tar.gz" "$SOURCE_DIR"        log "Backup completed"    else        log "Source directory not found: $SOURCE_DIR"    fi}main() {    log "Maintenance started"    check_disk    check_memory    check_service    rotate_logs    backup_app    log "Maintenance completed"}main
Run:
chmod +x maintenance.sh./maintenance.sh

32. Suggested Learning Path
Aap is order me practice karo:
Day 1: Linux basic commandsDay 2: Shell script basicsDay 3: Variables and inputDay 4: Arguments and exit codesDay 5: ConditionsDay 6: LoopsDay 7: FunctionsDay 8: Text processingDay 9: Error handlingDay 10: Cron jobsDay 11: Backup automationDay 12: Log analyzerDay 13: Service monitoringDay 14: Deployment scriptingDay 15: Docker scriptingDay 16: Final server maintenance project

Final Advice
Shell scripting me expert banne ke liye theory se zyada practice important hai.
Aapko ye 5 cheeze strongly aani chahiye:
1. Variables, arguments, input2. if-else, loops, functions3. grep, awk, sed, find4. error handling with set -euo pipefail5. real-world scripts: backup, logs, service check, deployment
Agar aap upar ke 25 tasks properly kar lete ho, to DevOps shell scripting ke liye aapka base strong ho jayega.