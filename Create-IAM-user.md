Absolutely, John! Here's the updated `README.md` version **without the JSON policy**, guiding you step-by-step to manually create the IAM policy through the AWS Console UI.

---

````markdown
# AWS IAM User Setup for S3 Backup Access

This guide walks you through creating an IAM user with limited access to an S3 bucket used for PostgreSQL database or log backups.

---

## ✅ Step 1: Create IAM User

1. Go to the **AWS Management Console → IAM → Users → Add user**
2. Set **User name**: `s3-backup-user`
3. Enable ✅ **Programmatic access** (for AWS CLI usage)
4. Click **Next: Permissions**

---

## ✅ Step 2: Set Custom Permissions

1. Choose **Attach policies directly**
2. Click **Create policy** (this opens in a new tab)

---

## ✅ Step 3: Create Custom Policy (Manual Steps)

1. In the **Create policy** screen:
   - Click the **Visual editor** tab
   - Under **Service**, choose `S3`
   - Under **Actions**, check:
     - ✅ `ListBucket`
     - ✅ `PutObject`
     - ✅ `GetObject`
     - ✅ `DeleteObject`
   - Under **Resources**:
     - For **bucket**, click **Add ARN**, then enter:
       - Bucket name: `pg-db-backup-bucket`
     - For **objects**, click **Add ARN**, then enter:
       - Bucket name: `pg-db-backup-bucket`
       - Object name: `*`

2. Click **Next**, then **Review**
3. Name the policy: `S3BackupPolicy`
4. Click **Create policy**

---

## ✅ Step 4: Attach Policy to User

1. Go back to the **Add user** tab
2. Click **Refresh**, then search for `S3BackupPolicy`
3. Select it, and proceed with **Next → Create user**

---

## ✅ Step 5: Download AWS Credentials

After creation, download the `.csv` file containing:

- **AWS Access Key ID**
- **AWS Secret Access Key**

---

## ✅ Step 6: Configure AWS CLI on the Instance

Run on the EC2 instance:

```bash
aws configure
````

Enter the values from your downloaded credentials:

* **Access Key ID**
* **Secret Access Key**
* **Region**: `ap-south-1`
* **Output format**: `json` or leave blank

> Make credentials available to cron jobs:

```bash
sudo mkdir -p /root/.aws
sudo cp ~/.aws/credentials /root/.aws/credentials
```

---

## ✅ Step 7: Test Upload

```bash
echo "test" > test.txt
aws s3 cp test.txt s3://pg-db-backup-bucket/
```

---

✅ Done! Your IAM user can now upload PostgreSQL backups to the S3 bucket securely.

```

Let me know if you'd like a second user for **read-only** access or alerts, or to convert this guide to a `.txt` or `.md` file for upload to your repo.
```
