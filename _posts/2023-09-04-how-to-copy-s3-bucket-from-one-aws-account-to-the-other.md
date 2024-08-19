---
layout: post
title:  "How to copy S3 bucket from one AWS account to the other"
---

# How to copy S3 bucket from one AWS account to the other

1. Allow `ListBucket`, `GetObjectTagging`, and `GetObject` access on the source bucket for the destination account. This 
   can be done by adding this statement to the source bucket's "Bucket policy";
   ```json
   {
       "Sid": "Any_name_to_identify_the_purpose_of_this_statement",
       "Effect": "Allow",
       "Principal": {
           "AWS": "arn:aws:iam::DESTINATION_AWS_ACCOUNT_NUMBER:root"
       },
       "Action": [
           "s3:ListBucket",
           "s3:GetObject",
           "s3:GetObjectTagging"
       ],
       "Resource": [
           "arn:aws:s3:::SOURCE_BUCKET_NAME/*",
           "arn:aws:s3:::SOURCE_BUCKET_NAME"
       ]
    }
    ```
    Replace bucket names and the account number in policy file before applying.
2. Log in to the AWS console using the destination account's credentials.
3. Create the destination bucket if it is not created already.
4. Open AWS CloudShell and issue this command; OR use AWS CLI if already installed.
   ```shell
   aws s3 sync s3://SOURCE_BUCKET_NAME s3://DESTINATION_BUCKET_NAME
   ```
5. This command may take some time depending upon the amount of data being transferred. Once the command execution is
   complete, the transfer is done. If for some reason the command execution gets interrupted, it can be repeated to copy 
   over the missing files. https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html