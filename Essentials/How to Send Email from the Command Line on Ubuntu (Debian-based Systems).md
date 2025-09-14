# 📧 How to Send Email from the Command Line on Ubuntu (Debian-based Systems)

Sending emails from the command line is a useful skill for developers, system administrators, and anyone automating tasks on Linux. This tutorial walks you through several ways to send emails from the command line on Debian-based systems like Ubuntu.

---

## 🛠️ Prerequisites

Before you begin, make sure you have:

- A Debian-based Linux system (e.g. Ubuntu)
- Access to the terminal
- A working internet connection
- Access to an external SMTP server (like Gmail) if required

## 📦 Method 1: Using `mail` Command (mailutils)

### 🔧 Step 1: Install `mailutils`

```bash
sudo apt update
sudo apt install mailutils
```

This will also install Postfix as a dependency. You may need to configure it during installation. When prompted, choose Internet Site and set your system mail name as your domain or hostname - for example: yourdomain.com or localhost.

### 📤 Step 2: Send a Simple Email

```bash
echo "This is the body of the email" | mail -s "Subject of the Email" recipient@example.com
```

- `-s` specifies the email subject  
- `recipient@example.com` is the email address of the recipient

### 📎 Step 3: Send Email with Attachment

```bash
echo "Please find the attachment." | mail -s "Subject with Attachment" -A /path/to/file.txt recipient@example.com
```

- `-A` adds an attachment to the email

## 📦 Method 2: Using `sendmail`

`sendmail` is a powerful mail transfer agent (MTA) that can be used to send emails directly or through scripts.

### 🔧 Step 1: Install `sendmail`

```bash
sudo apt update
sudo apt install sendmail
```

### 📤 Step 2: Send a Simple Email

```bash
sendmail recipient@example.com
```

Then type the following content:

```
Subject: Test Email
This is a test email sent using sendmail.
.
```

- The email ends when you type a single period `.` on a new line and press Enter.

## 📦 Method 3: Using `ssmtp` (Deprecated - Not Recommended)

⚠️ `ssmtp` is deprecated and no longer maintained. It has been replaced by `msmtp`. Use this method only if required for legacy support.

### 🔧 Step 1: Install `ssmtp`

```bash
sudo apt update
sudo apt install ssmtp
```

### ⚙️ Step 2: Configure `ssmtp`

Edit the configuration file:

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Example configuration:

```
root=your_email@gmail.com
mailhub=smtp.gmail.com:587
AuthUser=your_email@gmail.com
AuthPass=your_app_password
UseSTARTTLS=YES
```

Use an app-specific password if you're using Gmail with two-factor authentication.

### 📤 Step 3: Send an Email

```bash
echo "This is a test email" | ssmtp recipient@example.com
```

## 📦 Method 4: Using `mutt`

`mutt` is a command-line mail client suitable for sending emails with attachments and more.

### 🔧 Step 1: Install `mutt`

```bash
sudo apt update
sudo apt install mutt
```

### 📤 Step 2: Send an Email with Subject and Attachment

```bash
echo "Email body" | mutt -s "Email Subject" -a /path/to/file.txt -- recipient@example.com
```

- `-s` specifies the subject  
- `-a` specifies the attachment  
- `--` indicates the end of options before specifying the recipient

## 📦 Method 5: Using `msmtp` (Recommended SMTP Client)

`msmtp` is a lightweight SMTP client that's ideal for sending emails via external SMTP servers.

### 🔧 Step 1: Install `msmtp`

```bash
sudo apt update
sudo apt install msmtp msmtp-mta
```

### ⚙️ Step 2: Configure `msmtp`

Create a configuration file:

```bash
nano ~/.msmtprc
```

Paste the following configuration (example for Gmail):

```
defaults
auth on
tls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile ~/.msmtp.log

account gmail
host smtp.gmail.com
port 587
from your_email@gmail.com
user your_email@gmail.com
password your_app_password

account default : gmail
```

Make the config file readable only to your user:

```bash
chmod 600 ~/.msmtprc
```

### 📤 Step 3: Send an Email

```bash
echo "Hello from msmtp" | msmtp recipient@example.com
```

## 📎 Bonus: Sending HTML Emails

To send an HTML-formatted email using `sendmail`, use the following steps.

### 🧾 Step 1: Create the Email File

```bash
nano email.txt
```

Add this content:

```
Subject: HTML Email
Content-Type: text/html

<h1>This is an HTML Email</h1>
<p>This is a paragraph in HTML email.</p>
```

### 📤 Step 2: Send the Email

```bash
sendmail recipient@example.com < email.txt
```

---

## 🧪 Checking Email Logs

To check if emails were sent successfully:

```bash
tail -f /var/log/mail.log
```

If the log file doesn't exist, make sure that Postfix or Sendmail is properly installed and configured.

## ✅ Summary

| Method   | Tool       | Features                          | Recommended Use               |
|----------|------------|-----------------------------------|-------------------------------|
| Method 1 | mail       | Simple CLI utility                | Quick emails without setup    |
| Method 2 | sendmail   | Full MTA                          | Internal mail delivery        |
| Method 3 | ssmtp      | Simple SMTP client (deprecated)   | Legacy systems only           |
| Method 4 | mutt       | Text-based email client           | Emails with attachments       |
| Method 5 | msmtp      | Lightweight SMTP client           | Sending via Gmail/SMTP        |

---

## 🛡️ Security Tips

- Never store plain-text passwords in configuration files if avoidable  
- Use environment variables or encrypted secrets  
- For Gmail, use app passwords instead of your main account password  
- Always restrict permissions on config files using `chmod 600`

---

## 🔚 Conclusion

Sending email from the command line on Ubuntu is a powerful way to automate notifications, reports, or alerts. Whether you prefer the simplicity of `mail`, the flexibility of `sendmail`, or the security of SMTP clients like `msmtp`, there is a tool that fits your needs.

Happy emailing! 📬