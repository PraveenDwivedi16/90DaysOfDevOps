# Day 19 – Shell Scripting Project: Log Rotation, Backup & Crontab

## Task Overview

In Day 19, I applied shell scripting concepts from Day 16 to Day 18 in real-world DevOps mini projects.

This task focuses on:

- Log rotation script
- Server backup script
- Crontab scheduling
- Scheduled maintenance script
- Error handling
- Arguments
- Functions
- Real-world automation patterns

## Folder Structure

2026/
└── day-19/
    ├── log_rotate.sh
    ├── backup.sh
    ├── maintenance.sh
    └── day-19-project.md

# Task 1: Log Rotation Script

## Requirement

Create `log_rotate.sh` that:

- Takes a log directory as an argument
- Compresses `.log` files older than 7 days using `gzip`
- Deletes `.gz` files older than 30 days
- Prints how many files were compressed and deleted
- Exits with an error if the directory does not exist

## Script: log_rotate.sh

#!/bin/bash
set -euo pipefail

if [ $# -ne 1 ]; then
    echo "Usage: ./log_rotate.sh <log_directory>"
    exit 1
fi

LOG_DIR=$1

if [ ! -d "$LOG_DIR" ]; then
    echo "Error: Directory '$LOG_DIR' does not exist."
    exit 1
fi

echo "Starting log rotation for directory: $LOG_DIR"

COMPRESS_COUNT=$(find "$LOG_DIR" -type f -name "*.log" -mtime +7 | wc -l)

find "$LOG_DIR" -type f -name "*.log" -mtime +7 -exec gzip {} \;

DELETE_COUNT=$(find "$LOG_DIR" -type f -name "*.gz" -mtime +30 | wc -l)

find "$LOG_DIR" -type f -name "*.gz" -mtime +30 -delete

echo "Log rotation completed."
echo "Files compressed: $COMPRESS_COUNT"
echo "Compressed files deleted: $DELETE_COUNT"

## Explanation

set -euo pipefail

This makes the script safer.

- `set -e` exits when a command fails
- `set -u` exits when an undefined variable is used
- `set -o pipefail` fails the pipeline if any command inside the pipeline fails

if [ $# -ne 1 ]; then

This checks whether exactly one argument is passed.

The script expects one argument:

log directory path
Example:

./log_rotate.sh /var/log/myapp

if [ ! -d "$LOG_DIR" ]; then

This checks whether the given directory exists.

If the directory does not exist, the script exits with an error.

find "$LOG_DIR" -type f -name "*.log" -mtime +7

This finds `.log` files older than 7 days.

-exec gzip {} \;

This compresses each old `.log` file using `gzip`.

Example:

app.log

becomes:

app.log.gz

find "$LOG_DIR" -type f -name "*.gz" -mtime +30 -delete

This deletes `.gz` files older than 30 days.

## Commands Used

vim log_rotate.sh
chmod +x log_rotate.sh
./log_rotate.sh /var/log/myapp

## Sample Output

Starting log rotation for directory: /var/log/myapp
Log rotation completed.
Files compressed: 4
Compressed files deleted: 2

## Output If Directory Does Not Exist

Error: Directory '/var/log/myapp' does not exist.

# Task 2: Server Backup Script

## Requirement

Create `backup.sh` that:

- Takes a source directory and backup destination as arguments
- Creates a timestamped `.tar.gz` archive
- Verifies the archive was created successfully
- Prints archive name and size
- Deletes backups older than 14 days from the destination
- Handles errors
- Exits if source directory does not exist

## Script: backup.sh

#!/bin/bash
set -euo pipefail

if [ $# -ne 2 ]; then
    echo "Usage: ./backup.sh <source_directory> <backup_destination>"
    exit 1
fi

SOURCE_DIR=$1
BACKUP_DEST=$2

if [ ! -d "$SOURCE_DIR" ]; then
    echo "Error: Source directory '$SOURCE_DIR' does not exist."
    exit 1
fi

if [ ! -d "$BACKUP_DEST" ]; then
    echo "Backup destination does not exist. Creating: $BACKUP_DEST"
    mkdir -p "$BACKUP_DEST"
fi

TIMESTAMP=$(date +%Y-%m-%d)
ARCHIVE_NAME="backup-$TIMESTAMP.tar.gz"
ARCHIVE_PATH="$BACKUP_DEST/$ARCHIVE_NAME"

echo "Starting backup..."
echo "Source: $SOURCE_DIR"
echo "Destination: $ARCHIVE_PATH"

tar -czf "$ARCHIVE_PATH" "$SOURCE_DIR"

if [ -f "$ARCHIVE_PATH" ]; then
    ARCHIVE_SIZE=$(du -h "$ARCHIVE_PATH" | awk '{print $1}')
    echo "Backup created successfully."
    echo "Archive name: $ARCHIVE_NAME"
    echo "Archive size: $ARCHIVE_SIZE"
else
    echo "Error: Backup archive was not created."
    exit 1
fi

echo "Deleting backups older than 14 days from $BACKUP_DEST"

find "$BACKUP_DEST" -type f -name "backup-*.tar.gz" -mtime +14 -delete

echo "Backup cleanup completed."

## Explanation

if [ $# -ne 2 ]; then

This script expects two arguments:

1. Source directory
2. Backup destination directory

Example:
./backup.sh /var/www/html /backup
if [ ! -d "$SOURCE_DIR" ]; then

This checks if the source directory exists.

If source does not exist, backup cannot continue.

mkdir -p "$BACKUP_DEST"
This creates the backup destination if it does not already exist.

TIMESTAMP=$(date +%Y-%m-%d) 
This creates a date-based timestamp.

Example:
2026-02-08
ARCHIVE_NAME="backup-$TIMESTAMP.tar.gz"

This creates archive name like:

backup-2026-02-08.tar.gz

tar -czf "$ARCHIVE_PATH" "$SOURCE_DIR"

This creates a compressed backup archive.

Meaning:

- `tar` creates archive
- `-c` creates new archive
- `-z` compresses with gzip
- `-f` specifies file name

du -h "$ARCHIVE_PATH"

This prints archive size in human-readable format.

find "$BACKUP_DEST" -type f -name "backup-*.tar.gz" -mtime +14 -delete

This deletes old backups older than 14 days.

## Commands Used
vim backup.sh
chmod +x backup.sh
./backup.sh /var/www/html /backup

## Sample Output

Starting backup...
Source: /var/www/html
Destination: /backup/backup-2026-02-08.tar.gz
Backup created successfully.
Archive name: backup-2026-02-08.tar.gz
Archive size: 25M
Deleting backups older than 14 days from /backup
Backup cleanup completed.

## Output If Source Directory Does Not Exist

Error: Source directory '/var/www/html' does not exist.

# Task 3: Crontab

## What is Crontab?

Crontab is used to schedule commands or scripts to run automatically at specific times.

Example:
crontab -l
This shows currently scheduled cron jobs.

## Check Current Crontab
crontab -l

## Possible Output If No Cron Jobs Exist
no crontab for ubuntu
or:

no crontab for user

## Cron Syntax

* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0-7)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)

## Cron Field Explanation

| Field | Meaning | Example |
|---|---|---|
| Minute | 0-59 | `0` |
| Hour | 0-23 | `2` |
| Day of Month | 1-31 | `*` |
| Month | 1-12 | `*` |
| Day of Week | 0-7 | `0` or `7` means Sunday |

## Cron Entry 1: Run log_rotate.sh Every Day at 2 AM
0 2 * * * /home/ubuntu/2026/day-19/log_rotate.sh /var/log/myapp >> /var/log/log_rotate_cron.log 2>&1

Meaning:
Run every day at 2:00 AM

## Cron Entry 2: Run backup.sh Every Sunday at 3 AM

0 3 * * 0 /home/ubuntu/2026/day-19/backup.sh /var/www/html /backup >> /var/log/backup_cron.log 2>&1

Meaning:

Run every Sunday at 3:00 AM

## Cron Entry 3: Run Health Check Script Every 5 Minutes

Example:

*/5 * * * * /home/ubuntu/2026/day-19/health_check.sh >> /var/log/health_check.log 2>&1

Meaning:

Run every 5 minutes

## Important Note

Do not apply cron jobs directly unless paths are correct.

First check script path:
pwd
Then use full absolute path in cron.

## How to Edit Crontab

crontab -e

# Task 4: Combine — Scheduled Maintenance Script

## Requirement

Create `maintenance.sh` that:

- Calls the log rotation function
- Calls the backup function
- Logs all output to `/var/log/maintenance.log` with timestamps
- Write cron entry to run it daily at 1 AM


## Script: maintenance.sh

#!/bin/bash
set -euo pipefail

LOG_DIR="/var/log/myapp"
SOURCE_DIR="/var/www/html"
BACKUP_DEST="/backup"
MAINTENANCE_LOG="/var/log/maintenance.log"

log_message() {
    local MESSAGE=$1
    echo "$(date '+%Y-%m-%d %H:%M:%S') : $MESSAGE" | tee -a "$MAINTENANCE_LOG"
}

rotate_logs() {
    log_message "Starting log rotation."

    if [ ! -d "$LOG_DIR" ]; then
        log_message "Error: Log directory '$LOG_DIR' does not exist."
        return 1
    fi

    local COMPRESS_COUNT
    COMPRESS_COUNT=$(find "$LOG_DIR" -type f -name "*.log" -mtime +7 | wc -l)

    find "$LOG_DIR" -type f -name "*.log" -mtime +7 -exec gzip {} \;

    local DELETE_COUNT
    DELETE_COUNT=$(find "$LOG_DIR" -type f -name "*.gz" -mtime +30 | wc -l)

    find "$LOG_DIR" -type f -name "*.gz" -mtime +30 -delete

    log_message "Log rotation completed. Compressed: $COMPRESS_COUNT, Deleted: $DELETE_COUNT"
}

run_backup() {
    log_message "Starting backup."

    if [ ! -d "$SOURCE_DIR" ]; then
        log_message "Error: Source directory '$SOURCE_DIR' does not exist."
        return 1
    fi

    if [ ! -d "$BACKUP_DEST" ]; then
        log_message "Backup destination does not exist. Creating: $BACKUP_DEST"
        mkdir -p "$BACKUP_DEST"
    fi

    local TIMESTAMP
    TIMESTAMP=$(date +%Y-%m-%d)

    local ARCHIVE_NAME
    ARCHIVE_NAME="backup-$TIMESTAMP.tar.gz"

    local ARCHIVE_PATH
    ARCHIVE_PATH="$BACKUP_DEST/$ARCHIVE_NAME"

    tar -czf "$ARCHIVE_PATH" "$SOURCE_DIR"

    if [ -f "$ARCHIVE_PATH" ]; then
        local ARCHIVE_SIZE
        ARCHIVE_SIZE=$(du -h "$ARCHIVE_PATH" | awk '{print $1}')

        log_message "Backup created successfully. Archive: $ARCHIVE_NAME, Size: $ARCHIVE_SIZE"
    else
        log_message "Error: Backup archive was not created."
        return 1
    fi

    find "$BACKUP_DEST" -type f -name "backup-*.tar.gz" -mtime +14 -delete

    log_message "Old backup cleanup completed."
}

main() {
    log_message "Scheduled maintenance started."

    rotate_logs
    run_backup

    log_message "Scheduled maintenance completed."
}

main

## Explanation

LOG_DIR="/var/log/myapp"
SOURCE_DIR="/var/www/html"
BACKUP_DEST="/backup"
MAINTENANCE_LOG="/var/log/maintenance.log"

These variables store paths used by the maintenance script.

log_message() {
    local MESSAGE=$1
    echo "$(date '+%Y-%m-%d %H:%M:%S') : $MESSAGE" | tee -a "$MAINTENANCE_LOG"
}

This function logs messages with timestamps.

Example:

2026-02-08 01:00:00 : Scheduled maintenance started.

rotate_logs()

This function performs log rotation.

It compresses old `.log` files and deletes old `.gz` files.

run_backup()

This function creates backup archive and deletes old backups.

main()

This function controls the full flow:

1. Start maintenance
2. Run log rotation
3. Run backup
4. Finish maintenance

## Commands Used

vim  maintenance.sh
chmod +x maintenance.sh
sudo ./maintenance.sh

## Sample Output
2026-02-08 01:00:00 : Scheduled maintenance started.
2026-02-08 01:00:00 : Starting log rotation.
2026-02-08 01:00:01 : Log rotation completed. Compressed: 3, Deleted: 1
2026-02-08 01:00:01 : Starting backup.
2026-02-08 01:00:05 : Backup created successfully. Archive: backup-2026-02-08.tar.gz, Size: 25M
2026-02-08 01:00:05 : Old backup cleanup completed.
2026-02-08 01:00:05 : Scheduled maintenance completed.

## Cron Entry: Run maintenance.sh Daily at 1 AM
0 1 * * * /home/ubuntu/2026/day-19/maintenance.sh >> /var/log/maintenance_cron.log 2>&1
Meaning:
Run every day at 1:00 AM

# Testing with Safe Demo Directories
If you do not want to test on real `/var/log` or `/var/www/html`, create demo folders.
mkdir -p /tmp/myapp-logs
mkdir -p /tmp/source-app
mkdir -p /tmp/backups
Create sample files:
echo "old log" > /tmp/myapp-logs/app.log
echo "test file" > /tmp/source-app/index.html

Run log rotation:
./log_rotate.sh /tmp/myapp-logs
Run backup:
./backup.sh /tmp/source-app /tmp/backups

# Cron Entries Written

## Run log_rotate.sh Every Day at 2 AM

0 2 * * * /home/ubuntu/2026/day-19/log_rotate.sh /var/log/myapp >> /var/log/log_rotate_cron.log 2>&1


## Run backup.sh Every Sunday at 3 AM

0 3 * * 0 /home/ubuntu/2026/day-19/backup.sh /var/www/html /backup >> /var/log/backup_cron.log 2>&1


## Run Health Check Script Every 5 Minutes

*/5 * * * * /home/ubuntu/2026/day-19/health_check.sh >> /var/log/health_check.log 2>&1


## Run maintenance.sh Daily at 1 AM

0 1 * * * /home/ubuntu/2026/day-19/maintenance.sh >> /var/log/maintenance_cron.log 2>&1


# What I Learned

## 1. Log Rotation Helps Manage Disk Space

I learned how to compress old log files and delete very old compressed logs using `find`, `gzip`, and `-mtime`.

This is important because logs can grow quickly and consume server disk space.

## 2. Backups Should Be Automated and Verified

I learned how to create timestamped `.tar.gz` backups using shell scripts.

I also learned that backup scripts should verify whether the archive was created successfully.


## 3. Crontab Automates Repeated Server Tasks

I learned how to schedule scripts using cron syntax.

With crontab, DevOps engineers can automate log rotation, backups, health checks, and maintenance jobs.

