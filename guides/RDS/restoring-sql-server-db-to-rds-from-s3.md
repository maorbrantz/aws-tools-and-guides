# Guide: Restoring SQL Server Database to RDS using S3

This guide details the complete process of backing up and restoring a SQL Server database to Amazon RDS using Amazon S3.

## Prerequisites
- Access to the source database
- AWS account with appropriate permissions
- AWS CLI installed and configured
- Access to an Amazon S3 bucket in the same region as RDS

## Step 1: Creating a Full Backup of the Source Database

### For Databases Smaller than 5 TiB:
```sql
USE [database_name]
GO

BACKUP DATABASE [database_name] TO
DISK = 'C:\Backup\backup_name.bak'
WITH NOFORMAT, NOINIT,
NAME = 'Full Backup', SKIP, NOREWIND, NOUNLOAD, STATS = 10
GO
```

### For Databases Larger than 5 TiB:
Split the backup into multiple files, each smaller than 5 TiB:
```sql
USE [database_name]
GO

BACKUP DATABASE [database_name] TO
DISK = 'C:\Backup\backup_name1.bak',
DISK = 'C:\Backup\backup_name2.bak',
DISK = 'C:\Backup\backup_name3.bak'
WITH NOFORMAT, NOINIT,
NAME = 'Full Backup', SKIP, NOREWIND, NOUNLOAD, STATS = 10
GO
```

## Step 2: Uploading Backup Files to Amazon S3

### Uploading a Single Backup File:
```bash
aws s3 cp C:\Backup\backup_name.bak s3://bucket-name/
```

### Uploading Multiple Backup Files:
```bash
aws s3 cp "C:\Backup" s3://bucket-name/ --recursive
```

## Step 3: Setting Up IAM Role

1. Create a new IAM Role named `sql-server-backup-restore`
2. Add the following Trust Relationship:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "rds.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
3. Attach the following Policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::bucket-name",
                "arn:aws:s3:::bucket-name/*"
            ]
        }
    ]
}
```

## Step 4: Configuring RDS Option Group

1. Open AWS Console and navigate to RDS
2. Select Option Groups and choose Create option group
3. Configure the following details:
   - Name: SQLServerrestore
   - Description: SQLServerrestore
   - Engine: sqlserver-se
   - Major engine version: Select appropriate version
4. Add the SQLSERVER_BACKUP_RESTORE option
5. Select the IAM Role created in the previous step

## Step 5: Restoring the Database

1. Attach the created Option Group to your RDS instance
2. Execute the following command for single file restore:
```sql
exec msdb.dbo.rds_restore_database
@restore_db_name='new_database_name',
@s3_arn_to_restore_from='arn:aws:s3:::bucket-name/backup_name.bak';
```

3. For multiple file restore:
```sql
exec msdb.dbo.rds_restore_database
@restore_db_name='new_database_name',
@s3_arn_to_restore_from='arn:aws:s3:::bucket-name/backup_name*';
```

4. Check restore status:
```sql
exec msdb.dbo.rds_task_status
[@db_name='new_database_name'],
[@task_id=task_number];
```

## Important Notes
- Ensure the S3 bucket is in the same region as the RDS instance
- Maximum supported backup size is 64 TiB
- For SQL Server Express Edition, maximum size is 10 GiB
- Recommended to perform the process during off-peak hours
- Ensure sufficient free space in the RDS instance

## Common Issues and Solutions
1. Permission Error: Verify IAM Role is properly configured and Policy contains required permissions
2. Size Error: Ensure backup doesn't exceed RDS limitations
3. Region Error: Verify S3 bucket and RDS instance are in the same region
