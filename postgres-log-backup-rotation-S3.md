# AWS PostgreSQL Logs Backup Setup to S3 (Cron-Based)

This guide sets up automatic PostgreSQL log backups to an S3 bucket using a shell script and cron. The AWS CLI is installed manually from the official zip.

---

## Step 1: Install AWS CLI (Manual ZIP method)

```bash
cd /tmp
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version  # Confirm installation
```

---

## Step 2: Install Dependencies

```bash
sudo apt update
sudo apt install zip postgresql-client -y
```

---

## Step 3: Create the S3 bucket (if not already created)

```bash
aws s3 mb s3://pg-log-backup-bucket-react --region ap-south-1
```

---

## Step 4: Create Backup Script

Create file `/opt/pg-log-backup.sh`:

```bash
#!/bin/bash

TIMESTAMP=$(date +"%Y-%m-%d_%H-%M")
BACKUP_DIR="/var/log/postgresql"
ZIP_FILE="/tmp/pg_logs_$TIMESTAMP.zip"
S3_BUCKET="s3://pg-log-backup-bucket-react"

echo "[INFO] Backup started at $(date)"
cd /
zip -r "$ZIP_FILE" "$BACKUP_DIR"

aws s3 cp "$ZIP_FILE" "$S3_BUCKET/"

# Optional: Delete ZIPs older than 7 days from S3
aws s3 ls "$S3_BUCKET/" | while read -r line; do
    CREATE_DATE=$(echo $line | awk '{print $1" "$2}')
    FILE_NAME=$(echo $line | awk '{print $4}')
    FILE_DATE=$(date -d "$CREATE_DATE" +%s)
    NOW_DATE=$(date +%s)
    AGE=$(( (NOW_DATE - FILE_DATE) / 86400 ))
    if [ $AGE -gt 7 ]; then
        echo "[INFO] Deleting $FILE_NAME from S3 (older than 7 days)"
        aws s3 rm "$S3_BUCKET/$FILE_NAME"
    fi
done
```

---

## Step 5: Set Permissions

```bash
sudo chmod +x /opt/pg-log-backup.sh
```

---

## Step 6: AWS Credentials Setup (for root user)

If using `cron` or `sudo`, root needs AWS credentials. Run:

```bash
sudo mkdir -p /root/.aws
sudo cp ~/.aws/credentials /root/.aws/credentials
```

📝 If `~/.aws/credentials` doesn't exist, run:

```bash
aws configure
```

Provide:

* **Access Key ID**
* **Secret Access Key**
* **Region**: ap-south-1
* **Output format**: json (or leave blank)

---

## Step 7: Set Up Cron Job

```bash
sudo crontab -e
```

Add:

```bash
* * * * * /opt/pg-log-backup.sh >> /var/log/pg_backup.log 2>&1
```

(This runs every minute for testing. Adjust as needed.)

---

## Step 8: Check Logs

```bash
sudo tail -f /var/log/pg_backup.log
```

---

✅ Done! Your system will now back up PostgreSQL logs to S3 every minute (adjustable) and auto-delete ZIPs older than 7 days.
