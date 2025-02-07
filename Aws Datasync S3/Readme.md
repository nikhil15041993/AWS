# AWS DataSync: Transfer Files Between S3 Buckets in Different AWS Accounts

This guide explains how to move files from an S3 bucket in **AWS Account A** to an S3 bucket in **AWS Account B** using **AWS DataSync**.

## Prerequisites
- **AWS Account A** (Source S3 Bucket)
- **AWS Account B** (Destination S3 Bucket)
- **IAM Permissions** for DataSync to access both S3 buckets
- **AWS DataSync Agent**

---

## Step 1: Grant Permissions in Account B (Destination)

1. **Log in to AWS Account B**.
2. Navigate to **S3 > Your Destination Bucket**.
3. Go to the **Permissions** tab.
4. Add the following **Bucket Policy**, replacing `ACCOUNT_A_ID` with Account A's AWS ID and `BUCKET_NAME_B` with the destination bucket name:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::ACCOUNT_A_ID:root"
         },
         "Action": [
           "s3:PutObject",
           "s3:ListBucket",
           "s3:GetObject"
         ],
         "Resource": [
           "arn:aws:s3:::BUCKET_NAME_B",
           "arn:aws:s3:::BUCKET_NAME_B/*"
         ]
       }
     ]
   }
   ```
5. Click **Save changes**.

---

## Step 2: Create a DataSync Agent in Account A

1. **Sign in to AWS Account A**.
2. Navigate to **AWS DataSync**.
3. Click **Create agent**.
4. Choose **Amazon S3** as the source.
5. Select the **VPC and Subnet** where the agent will run.
6. Click **Activate agent**.

---

## Step 3: Create the DataSync Source Location (S3 in Account A)

1. In **AWS Account A**, go to **AWS DataSync**.
2. Click **Create location**.
3. Select **Amazon S3**.
4. Choose the **source S3 bucket**.
5. Click **Create location**.

---

## Step 4: Create the DataSync Destination Location (S3 in Account B)

1. Click **Create location**.
2. Select **Amazon S3**.
3. Enter the **destination bucket name** from Account B.
4. Under **AWS Credentials**, select **Use a different AWS account**.
5. Enter **Account B's IAM Role ARN** (see Step 5 for creating this role).
6. Click **Create location**.

---

## Step 5: Create an IAM Role in Account B for DataSync

1. **Log in to AWS Account B**.
2. Go to **IAM > Roles**.
3. Click **Create role**.
4. Select **Another AWS account** and enter **Account A’s AWS Account ID**.
5. Attach the following **policy**:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "s3:PutObject",
           "s3:ListBucket",
           "s3:GetObject"
         ],
         "Resource": [
           "arn:aws:s3:::BUCKET_NAME_B",
           "arn:aws:s3:::BUCKET_NAME_B/*"
         ]
       }
     ]
   }
   ```

6. Click **Next**, name the role (e.g., `DataSyncS3Access`), and **Create role**.
7. Copy the **Role ARN** and use it in **Step 4**.

---

## Step 6: Create a DataSync Task

1. Open **AWS DataSync** in Account A.
2. Click **Create task**.
3. Select **Source Location** (S3 in Account A).
4. Select **Destination Location** (S3 in Account B).
5. Configure transfer settings:
   - **Transfer Mode**: Copy or Move.
   - **File filtering**: Specify any file filters if needed.
6. Click **Create task**.

---

## Step 7: Start the DataSync Task

1. Navigate to **AWS DataSync > Tasks**.
2. Select the task you created.
3. Click **Start** to initiate the file transfer.

---
