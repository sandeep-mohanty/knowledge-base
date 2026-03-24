# Bash Shell Scripting—DAY 1
**Complete Beginner-Friendly Guide — A Step-by-Step Bash Scripting Guide for Freshers & Non-Technical Learners**

### What is Bash shell scripting?
**Bash (Bourne Again Shell)** is a command-line interpreter used in **Linux and Unix systems**. **Shell scripting** allows you to write a set of commands in a file and execute them automatically.

**In Simple Words**
👉 Bash scripting is like **giving instructions to your computer in plain English-style commands** so it can work for you automatically.

### Why Learn Bash Shell Scripting?
**Problems It Solves**
✔ Repeating the same commands every day  
✔ Manual system maintenance  
✔ Time-consuming file handling  
✔ Human errors in repetitive tasks  
✔ Lack of automation  

**Who Should Learn This?**
* Freshers (IT / Non-IT)
* Students
* System Administrators
* DevOps Beginners
* Linux Users
* Support Engineers
* Career switchers

**No heavy programming knowledge required ✅**

### Pre-Requisites (Beginner Friendly)
* Basic Linux commands (`ls`, `cd`, `pwd`)
* Terminal usage
* Any text editor (`vi`, `nano`, `VSCode`)
* Linux system (Local / VM / Cloud)

> ⚠️ **Important Note**
> Please **test all scripts in a local or development environment first** before using them in production. Sometimes, when scripts are copied from servers to blogs, small mistakes can happen (missing flags, wrong commands, or formatting issues). Running untested scripts directly in production may cause service issues or data loss.
> 👉 **Always test locally → verify → then implement on production.**

---

## MODULE-WISE COMPLETE NOTES & PRACTICE

### Module 1: Introduction to Bash
**What is Shell?**
A **shell** acts as a **bridge between user and operating system**.

**Types of Shell**
* `sh` — Bourne Shell
* `bash` — Bourne Again Shell
* `ksh` — Korn Shell
* `zsh` — Z Shell

**Shell vs Terminal vs Command Line**


**First Bash Script (Hello World)**
```bash
#!/bin/bash
echo "Hello, Welcome to Bash Scripting!"
```
Make it executable:
```bash
devops tushar]# chmod +x hello.sh
devops tushar]# ./hello.sh
```

---

### Module 2: Variables & Data Types
**What is a Variable?**
Variables store values (text or numbers) to reuse data in scripts.

```bash
name="Tushar"
echo "Hello $name"
```

**Special Variables**


---

### Module 3: Basic Commands & I/O — Input & Output
Using Linux commands inside scripts to manage files and data.

**User Input Example**
```bash
echo "Enter your name:"
read name
echo "Welcome $name"
```

---

### Module 4: Conditional Statements
Run commands only when conditions are true.

```bash
age=$1

if [ "$age" -ge 18 ]
then
  echo "Eligible"
else
  echo "Not Eligible"
fi
```
✔ **Run it**
```bash
devops tushar]# sh file.txt 10

# Output
Not Eligible
```
Try:
```bash
devops tushar]# sh file.txt 20

# Output:
Eligible
```

**Decision making in scripts.**
Run commands only when conditions are true.
```bash
num=10
if [ $num -gt 5 ]
then
  echo "Number is greater than 5"
fi
```

⭐ **Important Best Practices**
1.  **Always quote variables**
    `[ "$age" -ge 18 ]`
    This prevents errors when variables are empty.
2.  **Add argument validation (recommended)**
```bash
if [ $# -ne 1 ]; then
  echo "Usage: $0 <age>"
  exit 1
fi
age=$1
if [ "$age" -ge 18 ]; then
  echo "Eligible"
else
  echo "Not Eligible"
fi
```



---

### Module 5: Loops
Repeating tasks automatically and avoid writing same command again and again.

**For Loop**
```bash
for i in 1 2 3 4 5
do
  echo "Number $i"
done
```
**# Output:**
```bash
devops tushar]# sh file.txt
Number 1
Number 2
Number 3
Number 4
Number 5
```

**While Loop**
```bash
count=1
while [ $count -le 5 ]
do
  echo $count
  ((count++))
done
```
**# Output:**
```bash
devops tushar]# sh file.txt
1
2
3
4
5
```

---

### Module 6: Functions
Reusable blocks of code to clean and organized scripts.

```bash
greet() {
  echo "Hello $1"
}
greet "World"
greet "Rahul"
greet "Vivek"
greet "Ahishek"
```
**# Output:**
```text
Hello World
Hello Rahul
Hello vivek
Hello ahishek
```

---

### Module 7: Command Line Arguments
Passing values when running script. Make scripts dynamic.

```bash
echo "Script Name: $0"
echo "First Arg: $1"
```
```bash
devops tushar]# ./script.sh Linux 
Script Name: script.sh
First Arg: Linux
```

---

### Module 8: Error Handling
Checking if command succeeded to avoid silent failures.

```bash
cp file1.txt file2.txt
if [ $? -ne 0 ]
then
  echo "Error copying file"
fi
```

---

### Module 9: String Manipulation
Working with text. Example : Parse names, logs, messages.

```bash
text="LinuxBash"
echo ${#text}
echo ${text:0:5}
```

---

### Module 10: Arrays
Store multiple values in one variable to handle lists easily.

```bash
fruits=("Apple" "Banana" "Mango")
echo ${fruits[1]}
echo ${fruits[0]}
echo ${fruits[2]}

cars=("BMW" "Audi" "Tata")
echo ${cars[0]}
echo ${cars[2]}
echo ${cars[1]}
```

---

### Module 11: File & Directory Operations
Checking files/folders. Automation and safety.

```bash
if [ -f "data.txt" ]
then
  echo "File exists"
fi

if [ -d "/home/user" ]
then
  echo "Directory exists"
fi
```

---

### Module 12: Text Processing Tools
Search and modify text. Log analysis, reports.

**grep Example**
`grep "error" app.log`

**awk Example**
`awk '{print $1,$3}' file.txt`

**sed Example**
`sed 's/old/new/g' file.txt`

---

### Module 13: Pipes & Filters
Connecting commands. Process data efficiently.

```bash
ps aux | grep root | wc -l
ls | wc -l
```

---

### Module 14: Scheduling with Cron
Run scripts automatically. Daily backups, reports.

```bash
crontab -e
0 2 * * * /home/backup.sh
```
✔ **Runs daily at 2 AM**

---

### Module 15: Exit Codes & Advanced Errors
Custom success/failure codes. Professional scripts.

```bash
exit 1
```

---

### Module 16: Advanced Functions
Functions with complex logic. Large scripts.

```bash
sum() {
  echo $(($1+$2))
}
sum 3 5
```

---

### Module 17: Regular Expressions
Pattern matching. Advanced searching.

```bash
grep -E "^[A-Z]" file.txt
```

---

### Module 18: Debugging & Optimization
Finding script errors. Fix issues faster.

```bash
set -x
```

---

### Module 19: Advanced File Handling
Handling large files. Performance.

```bash
while read line
do
 echo $line
done < file.txt
```

---

### Module 20: Networking & Web
Internet access in scripts. APIs, downloads.

```bash
curl https://example.com
```

---

### Module 21: Process Management
Handling running programs. System control.

```bash
ps aux | grep apache
```

---

### Module 22: Advanced Scripting Techniques
Professional scripting styles.

```bash
getopts "a:b:" opt
```

---

### Module 23: Security Considerations
Safe scripting. Avoid data leaks.

```bash
chmod 700 script.sh
```

---

### Module 24: System Administration Scripts
Admin automation.

```bash
df -h
```

---

### Module 25: Integration with Other Tools
Using Python, jq, DBs.

```bash
jq '.name' file.json
```

---

### Module 26: Writing Modular Scripts
Splitting scripts.

```bash
source common.sh
```

---

### Module 27: Performance Tuning
Faster scripts.

```bash
time ./script.sh
```

---

### Module 28: grep
```bash
grep -i "linux" file.txt
```

---

### Module 29: AWK
```bash
awk '{print $1,$2}' data.txt
```

---

### Module 30: sed
```bash
sed 's/old/new/g' file.txt
```

---

## Real-Life Use Cases
1.  **Daily Backup Script**
    `tar -czf backup.tar.gz /home/user`
2.  **Disk Monitoring**
    `df -h | grep /dev/sda1`
3.  **User Creation Automation**
    `useradd testuser`
4.  **Log File Analysis**
    `grep "ERROR" app.log`

---

## Linux Log Monitoring & Auto-Recovery Using AWK and Bash Scripts
**How to analyze server logs, detect abnormal traffic, and automatically restart services when RAM usage is high**

### What Is This?
This solution is a **Linux server monitoring and automation setup** using:
* `awk`, `sort`, `uniq` – for log analysis
* `date` – for time-based filtering
* `bash scripts` – for RAM monitoring and auto-restarting services
* `ps`, `pgrep`, `kill` – for process control

**Common Production Issues:**
* Sudden traffic spikes from specific IPs
* DDoS-like behavior or abusive clients
* High memory usage causing services to crash
* Manual restarts during off-hours
* Slow response due to overloaded services

**This solution helps you:**
✔ Identify top IPs hitting your server  
✔ Detect traffic in the last few minutes  
✔ Count requests per IP  
✔ Monitor RAM usage in real time  
✔ Automatically restart services when memory crosses a threshold  

---

### Log Analysis Using AWK (Access Logs)

**1. Find Top IP Addresses Hitting Your Server**
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -20
```
**What it does:** Extracts IP addresses, counts requests per IP, and shows top 20 IPs.  
✅ **Use case:** Detect bots, crawlers, or abusive IPs.

**2. Count Requests in the Last 5 Minutes**
```bash
awk -v now="$(date -d 'now - 5 minutes' +'%Y-%m-%d %H:%M:%S')" \
'{ if ($4 >= now) count[$1]++ } 
END { for (ip in count) print ip, count[ip] }' access.log
```
**What it does:** Filters logs from last 5 minutes and counts IP-wise traffic.  
✅ **Use case:** Real-time traffic surge detection.

**3. Extract Recent Logs into a Separate File**
```bash
awk -v now="$(date -d 'now - 5 minutes' +'%Y-%m-%d %H:%M:%S')" \
'{ if ($4 >= now) print }' access.log > recent.log
```
✅ **Use case:** Debug recent incidents quickly.

**4. Match Logs by Specific Time**
```bash
awk '{ if ($4 ~ /2023-11-28 10:45:../) count++ } END { print count }' access.log
```
✅ **Use case:** Incident analysis at a precise timestamp.

**5. Redis Key Monitoring**
```bash
KEYS * | wc -l
```
**What it does:** Counts total keys in Redis.  
✅ **Use case:** Identify memory pressure due to excessive keys.

**6. Process Control (Force Kill Example)**
```bash
pgrep -f "grafana-server forge" | xargs kill -9
```
✅ **Use case:** Instantly kill stuck or runaway processes.

---

### 7. Automated RAM Monitoring & Service Restart (Bash)

**Example 1: Restart Grafana if RAM > 35%**
```bash
#!/bin/bash

check_ram_usage() {
  free -m | awk '/Mem:/ {print $3/$2 * 100.0}' | cut -d. -f1
}
while true; do
  ram_usage=$(check_ram_usage)
  if [[ $ram_usage -gt 35 ]]; then
    echo "RAM exceeded 35%. Restarting Grafana..."
    sleep 2
  elif [[ $ram_usage -le 30 ]]; then
    exit 0
  fi
  sleep 1
done
```
✅ **Use case:** Lightweight protection for monitoring tools.

**Example 2: Auto-Restart User Service When RAM > 80%**
```bash
#!/bin/bash

check_ram_usage() {
  free -m | awk '/Mem:/ {print $3/$2 * 100.0}' | cut -d. -f1
}
start_service() {
  cd /data/Jio\ Midas/User_Services
  export DEPLOYMENT_ENV=prod
  nohup ./user-service > /dev/null &
}
stop_service() {
  ps aux | grep '[u]ser-service' | awk '{print $2}' | xargs kill -9
}
while true; do
  ram_usage=$(check_ram_usage)
  if [[ $ram_usage -gt 80 ]]; then
    stop_service
    sleep 5
    start_service
  fi
  sleep 60
done
```
✅ **Use case:** Prevent memory crashes in production services.