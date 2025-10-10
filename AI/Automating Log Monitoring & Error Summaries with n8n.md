# 📜 Tutorial: Monitoring .NET Application Logs with n8n for Specific Error Patterns

This tutorial walks you through creating an **n8n workflow** to:  
- Periodically check your .NET app logs.  
- Detect specific patterns (e.g. `ERROR`, `CRITICAL`, `OutOfMemory`, etc.).  
- Notify you when they appear via Slack/Email.

---

## 📋 Prerequisites

- Ubuntu host with Dockerized **n8n running** (see previous tutorials).  
- Your .NET app logs path, e.g. `/var/log/mydotnetapp/`.  
- A notification channel configured in n8n (Slack/Email/Discord).  

---

## 1️⃣ Step 1: Setup a Scheduled Trigger

Use a **Cron Node** in n8n:  
- Run every minute (or every 5 minutes if logs are huge).  

This keeps the polling lightweight while still near‑realtime.

---

## 2️⃣ Step 2: Identify the Latest Log File

Add a **Shell Node** to grab the newest log file:  

    ls -t /var/log/mydotnetapp/*.log | head -n 1

This outputs the most recent log file name (say `application-2024-07-11.log`).

---

## 3️⃣ Step 3: Extract the Log Content

Another **Shell Node** to read only the *latest lines* (to avoid re‑parsing giant logs):  

    tail -n 200 $(ls -t /var/log/mydotnetapp/*.log | head -n 1)

This grabs the last 200 lines from the newest log file. Adjust as needed.

---

## 4️⃣ Step 4: Analyze for Specific Patterns

Use a **Function Node** — here you’ll define the exact patterns you want to catch:

    const logText = $json["data"] || "";
    const lines = logText.split("\n");

    // Define what to look for
    const patterns = [
      /ERROR/i,
      /CRITICAL/i,
      /OutOfMemory/,
      /DbTimeout/,
      /Unhandled Exception/
    ];

    const matched = [];
    for (const line of lines) {
      for (const regex of patterns) {
        if (regex.test(line)) {
          matched.push(line);
          break; // no need to test other patterns once matched
        }
      }
    }

    return [{
      issueCount: matched.length,
      issues: matched.slice(0,10).join("\n") // include top 10 matched lines for preview
    }];

---

## 5️⃣ Step 5: Notify When Matches Found

Add an **IF Node** after the Function Node:  
- Condition: `issueCount > 0`.  

If true, send an alert:  

- **Slack Node**:  

    🚨 .NET Log Alert 🚨  
    Found {{$json.issueCount}} issues in logs.  
    Example lines:  
    {{$json.issues}}  

- OR **Email Node** with subject `[n8n ALERT] .NET log issues detected`.

---

## 6️⃣ Step 6: Avoid Duplicate Alerts (Optional)

Use n8n **static data** to store the last processed file position (e.g. line number or byte offset).  
This way you only report **new log entries**, not old repeated ones.  

For example:  
- Store the timestamp of the last Cron run.  
- Compare latest log modification time against stored timestamp.  
- Only parse lines newer than the last run.

---

## ⚡ Advanced Options

- Parse **structured logs** (if your .NET app outputs JSON logs) instead of regex scanning. Use n8n’s JSON processing nodes for richer filtering.  
- Add a **Daily Digest** with a Cron Node that sends a summary email of total warnings/errors for the day.  
- Instead of “tail”, parse logs into an external database (e.g. SQLite, Postgres, ELK) and let n8n query them.  

---

## 🎯 Wrap-Up

This workflow allows you to:  
- Continuously monitor .NET logs for *specific error types*.  
- Reduce noise by only matching patterns you define.  
- Get proactive alerts before users complain.  

No more endless `tail -f` sessions in SSH — n8n instantly becomes your assistant for log triage. 🧑‍💻🐕