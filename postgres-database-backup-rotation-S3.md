Perfect, John! Here's a `README.md` file tailored to your request for **PostgreSQL Database Backup (not logs)** to S3 using a cron-based setup.

---

````markdown
# AWS PostgreSQL Database Backup to S3 (Cron-Based)

This guide helps you set up **automated PostgreSQL database dumps** and upload them to an **Amazon S3 bucket**, including **automated rotation** to delete backups older than 7 days.

---

## ✅ Step 1: Install AWS CLI (ZIP method)

```bash
cd /tmp
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version  # Confirm installation
````

---

## ✅ Step 2: Install Dependencies

```bash
sudo apt update
sudo apt install zip postgresql-client -y
```

---

## ✅ Step 3: Create the S3 Bucket (if needed)

```bash
aws s3 mb s3://pg-db-backup-bucket --region ap-south-1
```

---

## ✅ Step 4: Create the Backup Script

Create file `/opt/pg-db-backup.sh`:

```bash
#!/bin/bash

# === Config ===
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M")
DB_NAME="your_db_name"
DB_USER="your_db_user"
BACKUP_DIR="/tmp"
DUMP_FILE="$BACKUP_DIR/${DB_NAME}_$TIMESTAMP.sql"
ZIP_FILE="$BACKUP_DIR/${DB_NAME}_$TIMESTAMP.zip"
S3_BUCKET="s3://pg-db-backup-bucket"

# === Dump the DB ===
echo "[INFO] Starting DB backup at $(date)"
pg_dump -U "$DB_USER" "$DB_NAME" > "$DUMP_FILE"

# === Compress the dump ===
zip "$ZIP_FILE" "$DUMP_FILE"
rm "$DUMP_FILE"

# === Upload to S3 ===
aws s3 cp "$ZIP_FILE" "$S3_BUCKET/"
echo "[INFO] Uploaded $ZIP_FILE to $S3_BUCKET"

# === Cleanup local ZIP ===
rm "$ZIP_FILE"

# === Optional: Delete old files from S3 (older than 7 days) ===
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

> Replace `your_db_name` and `your_db_user` accordingly.

---

## ✅ Step 5: Set Script Permissions

```bash
sudo chmod +x /opt/pg-db-backup.sh
```

---

## ✅ Step 6: AWS Credentials for Root

If using cron or `sudo`, AWS credentials must be available for root:

```bash
sudo mkdir -p /root/.aws
sudo cp ~/.aws/credentials /root/.aws/credentials
```

If you don't have credentials configured, run:

```bash
aws configure
```

---

## ✅ Step 7: Setup Cron Job

```bash
sudo crontab -e
```

Add this line to run the backup every hour:

```bash
0 * * * * /opt/pg-db-backup.sh >> /var/log/pg_db_backup.log 2>&1
```

---

## ✅ Step 8: Monitor Logs

```bash
sudo tail -f /var/log/pg_db_backup.log
```

---

✅ **Done!** Your PostgreSQL database is now backed up hourly to S3, and backups older than 7 days are automatically deleted.

```

