Task 1: Input and Validation
Requirement

The script should:

Accept the path to a log file as a command-line argument
Exit with a clear error message if no argument is provided
Exit with a clear error message if the file does not exist
Code
if [ $# -ne 1 ]; then
    echo "Error: No log file provided."
    echo "Usage: ./log_analyzer.sh <log_file_path>"
    exit 1
fi

LOG_FILE=$1

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: File '$LOG_FILE' does not exist."
    exit 1
fi
Explanation

$# gives the total number of arguments passed to the script.

If no argument is provided, the script prints a usage message and exits.

-f checks whether the given path exists and is a regular file.

Task 2: Error Count
Requirement

Count the total number of lines containing the keyword ERROR or Failed.

Code
ERROR_COUNT=$(grep -Ei "ERROR|Failed" "$LOG_FILE" | wc -l)
Explanation

grep -Ei searches case-insensitively using extended regex.

Pattern:

ERROR|Failed

means match either ERROR or Failed.

wc -l counts the number of matching lines.

Output Example
Total error count: 10
Task 3: Critical Events
Requirement

Search for lines containing the keyword CRITICAL.

Print those lines with line numbers.

Code
CRITICAL_EVENTS=$(grep -n "CRITICAL" "$LOG_FILE" || true)

if [ -n "$CRITICAL_EVENTS" ]; then
    echo "$CRITICAL_EVENTS" | while IFS=: read -r LINE_NO LINE_CONTENT
    do
        echo "Line $LINE_NO: $LINE_CONTENT"
    done
else
    echo "No critical events found."
fi
Explanation

grep -n prints line numbers with matching lines.

Example output from grep -n:

7:2026-02-11 09:07:12 CRITICAL Disk space below threshold

The script formats this as:

Line 7: 2026-02-11 09:07:12 CRITICAL Disk space below threshold

|| true is used because with set -e, the script would exit if grep finds no match.

Task 4: Top 5 Error Messages
Requirement

Extract all lines containing ERROR, identify the top 5 most common error messages, and display them with occurrence count.

Code
TOP_ERRORS=$(grep "ERROR" "$LOG_FILE" | awk '
{
    for (i=1; i<=NF; i++) {
        if ($i == "ERROR") {
            for (j=i+1; j<=NF; j++) {
                printf "%s ", $j
            }
            printf "\n"
        }
    }
}' | sed 's/[[:space:]]*$//' | sort | uniq -c | sort -rn | head -5 || true)
Explanation

This command pipeline does the following:

Command	Purpose
grep "ERROR"	Finds lines containing ERROR
awk	Extracts message text after the ERROR keyword
sed	Removes trailing spaces
sort	Sorts messages
uniq -c	Counts duplicate messages
sort -rn	Sorts count in descending order
head -5	Shows top 5 results
Output Example
      3 Connection timed out
      2 File not found
      1 Permission denied
      1 Out of memory
      1 Disk I/O error
Task 5: Summary Report
Requirement

Generate a summary report named:

log_report_<date>.txt

Example:

log_report_2026-02-11.txt

The report should include:

Date of analysis
Log file name
Total lines processed
Total error count
Top 5 error messages
Critical events with line numbers
Code
ANALYSIS_DATE=$(date +%Y-%m-%d)
REPORT_FILE="log_report_${ANALYSIS_DATE}.txt"
TOTAL_LINES=$(wc -l < "$LOG_FILE")

Report generation:

{
    echo "Log Analysis Summary Report"
    echo "==========================="
    echo
    echo "Date of Analysis: $ANALYSIS_DATE"
    echo "Log File Name: $LOG_FILE"
    echo "Total Lines Processed: $TOTAL_LINES"
    echo "Total Error Count: $ERROR_COUNT"
} > "$REPORT_FILE"
Explanation

date +%Y-%m-%d creates the date format used in the report filename.

wc -l counts total lines in the log file.

> writes output to the report file.

Task 6: Optional Archive Processed Logs
Requirement

Add a feature to:

Create an archive/ directory if it does not exist
Move the processed log file into archive/
Print confirmation message
Code
read -p "Do you want to archive the processed log file? (y/n): " ARCHIVE_CHOICE

if [ "$ARCHIVE_CHOICE" = "y" ]; then
    mkdir -p archive
    mv "$LOG_FILE" archive/
    echo "Processed log file moved to archive/"
elif [ "$ARCHIVE_CHOICE" = "n" ]; then
    echo "Archive skipped."
else
    echo "Invalid choice. Archive skipped."
fi
Explanation

mkdir -p archive creates the archive directory if it does not exist.

mv "$LOG_FILE" archive/ moves the processed log file into the archive directory.

Complete Script: log_analyzer.sh
#!/bin/bash
set -euo pipefail

if [ $# -ne 1 ]; then
    echo "Error: No log file provided."
    echo "Usage: ./log_analyzer.sh <log_file_path>"
    exit 1
fi

LOG_FILE=$1

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: File '$LOG_FILE' does not exist."
    exit 1
fi

ANALYSIS_DATE=$(date +%Y-%m-%d)
REPORT_FILE="log_report_${ANALYSIS_DATE}.txt"
TOTAL_LINES=$(wc -l < "$LOG_FILE")
ERROR_COUNT=$(grep -Ei "ERROR|Failed" "$LOG_FILE" | wc -l)

echo "Analyzing log file: $LOG_FILE"
echo "Total lines processed: $TOTAL_LINES"
echo "Total error count: $ERROR_COUNT"
echo

echo "--- Critical Events ---"
CRITICAL_EVENTS=$(grep -n "CRITICAL" "$LOG_FILE" || true)

if [ -n "$CRITICAL_EVENTS" ]; then
    echo "$CRITICAL_EVENTS" | while IFS=: read -r LINE_NO LINE_CONTENT
    do
        echo "Line $LINE_NO: $LINE_CONTENT"
    done
else
    echo "No critical events found."
fi

echo
echo "--- Top 5 Error Messages ---"

TOP_ERRORS=$(grep "ERROR" "$LOG_FILE" | awk '
{
    for (i=1; i<=NF; i++) {
        if ($i == "ERROR") {
            for (j=i+1; j<=NF; j++) {
                printf "%s ", $j
            }
            printf "\n"
        }
    }
}' | sed 's/[[:space:]]*$//' | sort | uniq -c | sort -rn | head -5 || true)

if [ -n "$TOP_ERRORS" ]; then
    echo "$TOP_ERRORS"
else
    echo "No ERROR messages found."
fi

{
    echo "Log Analysis Summary Report"
    echo "==========================="
    echo
    echo "Date of Analysis: $ANALYSIS_DATE"
    echo "Log File Name: $LOG_FILE"
    echo "Total Lines Processed: $TOTAL_LINES"
    echo "Total Error Count: $ERROR_COUNT"
    echo

    echo "--- Top 5 Error Messages ---"
    if [ -n "$TOP_ERRORS" ]; then
        echo "$TOP_ERRORS"
    else
        echo "No ERROR messages found."
    fi

    echo
    echo "--- Critical Events ---"
    if [ -n "$CRITICAL_EVENTS" ]; then
        echo "$CRITICAL_EVENTS" | while IFS=: read -r LINE_NO LINE_CONTENT
        do
            echo "Line $LINE_NO: $LINE_CONTENT"
        done
    else
        echo "No critical events found."
    fi
} > "$REPORT_FILE"

echo
echo "Summary report generated: $REPORT_FILE"

read -p "Do you want to archive the processed log file? (y/n): " ARCHIVE_CHOICE

if [ "$ARCHIVE_CHOICE" = "y" ]; then
    mkdir -p archive
    mv "$LOG_FILE" archive/
    echo "Processed log file moved to archive/"
elif [ "$ARCHIVE_CHOICE" = "n" ]; then
    echo "Archive skipped."
else
    echo "Invalid choice. Archive skipped."
fi
Sample Log File
File: sample_log.log
2026-02-11 09:00:01 INFO Server started successfully
2026-02-11 09:02:10 ERROR Connection timed out
2026-02-11 09:03:22 INFO User login successful
2026-02-11 09:04:15 ERROR File not found
2026-02-11 09:05:45 Failed to connect to database
2026-02-11 09:06:30 ERROR Permission denied
2026-02-11 09:07:12 CRITICAL Disk space below threshold
2026-02-11 09:08:40 ERROR Connection timed out
2026-02-11 09:09:50 WARNING High memory usage
2026-02-11 09:10:05 ERROR Disk I/O error
2026-02-11 09:11:18 ERROR Connection timed out
2026-02-11 09:12:30 Failed authentication attempt
2026-02-11 09:13:40 CRITICAL Database connection lost
2026-02-11 09:14:20 ERROR File not found
2026-02-11 09:15:33 ERROR Out of memory
Commands Used
Make script executable
chmod +x log_analyzer.sh
Run script
./log_analyzer.sh sample_log.log
Sample Output
Analyzing log file: sample_log.log
Total lines processed: 15
Total error count: 10

--- Critical Events ---
Line 7: 2026-02-11 09:07:12 CRITICAL Disk space below threshold
Line 13: 2026-02-11 09:13:40 CRITICAL Database connection lost

--- Top 5 Error Messages ---
      3 Connection timed out
      2 File not found
      1 Permission denied
      1 Out of memory
      1 Disk I/O error

Summary report generated: log_report_2026-02-11.txt
Do you want to archive the processed log file? (y/n): n
Archive skipped.
Generated Report Example
File: log_report_2026-02-11.txt
Log Analysis Summary Report
===========================

Date of Analysis: 2026-02-11
Log File Name: sample_log.log
Total Lines Processed: 15
Total Error Count: 10

--- Top 5 Error Messages ---
      3 Connection timed out
      2 File not found
      1 Permission denied
      1 Out of memory
      1 Disk I/O error

--- Critical Events ---
Line 7: 2026-02-11 09:07:12 CRITICAL Disk space below threshold
Line 13: 2026-02-11 09:13:40 CRITICAL Database connection lost
Tools and Commands Used
1. grep

Used to search for matching text.

Example:

grep "ERROR" sample_log.log

Used for:

Finding ERROR
Finding Failed
Finding CRITICAL
2. grep -n

Used to print matching lines with line numbers.

Example:

grep -n "CRITICAL" sample_log.log
3. grep -Ei

Used for extended and case-insensitive matching.

Example:

grep -Ei "ERROR|Failed" sample_log.log
4. wc -l

Used to count lines.

Example:

wc -l < sample_log.log
5. awk

Used to extract the actual error message after the ERROR keyword.

6. sed

Used to clean trailing spaces.

7. sort

Used to sort error messages.

8. uniq -c

Used to count duplicate error messages.

9. head -5

Used to show only top 5 error messages.

10. date

Used to generate report filename with current date.

Example:

date +%Y-%m-%d
11. mkdir -p

Used to create archive directory if it does not exist.

12. mv

Used to move processed log file into archive directory.

What I Learned
1. Log Analysis Can Be Automated Using Bash

I learned how to scan log files automatically and extract useful information like errors, failures, and critical events.

2. Linux Commands Can Be Combined Using Pipelines

I learned how to combine commands like:

grep
awk
sort
uniq
head

to process and summarize log data.

3. Report Generation Is Useful for System Administration

I learned how to generate a daily summary report using shell scripting.

This is useful for DevOps engineers and system administrators who need regular visibility into server issues.
