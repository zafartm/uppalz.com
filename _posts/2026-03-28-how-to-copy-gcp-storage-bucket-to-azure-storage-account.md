---
layout: post
title:  "Copy GCP storage bucket to Azure storage account"
---

# How to copy GCP storage bucket to Azure storage account

## 1. Install requisite tools
1. az cli (https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt)
2. azcopy cli (https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10#download-azcopy)

## 2. Login to Azure CLI
Issue this command then following onscreen instructions;
```shell
az login
```

If working on an Azure VM that can use identity based login, then do;
```shell
az login --identity
```

## 3. Get GCP credentials and the source bucket name

Download the JSON credentials file for a service account that has permissions to read from the source bucket. Note that service account is required. `xcopy` doesn't recognise anyother form of GCP credentials.

Once the file is downloaded, set this env variable;
```shell
export GOOGLE_APPLICATION_CREDENTIALS="path-to-service-account-secrets-file.json"
```

Also set the source buket name in the env;
```shell
export BUCKET_NAME=source-bucket-in-gcp-storage
```

## 4. Get Azure credentials (SAS token) for the target storage account

Set target storage account name in the env.
```shell
export STORAGE_ACCOUNT_NAME=target-storage-account-name
```

Decide an expiry date for the sas token, then generate it. Tokan will remain valid till this date;
```shell
export SAS_EXPIRY=2026-03-31
export SAS_TOKEN=$(
   az storage account generate-sas \
      --expiry $SAS_EXPIRY \
      --account-name $STORAGE_ACCOUNT_NAME \
      --services b --resource-types co --permissions acuwrl \
      --https-only \
      --output tsv)
```

##  5. Copy the data from GCP to Azure
Issue this command to do the data copy.
```shell
azcopy copy --recursive=true \
      https://storage.cloud.google.com/$BUCKET_NAME \
      https://$STORAGE_ACCOUNT_NAME.blob.core.windows.net?$SAS_TOKEN
```

This may take a while to complete depending upon the amount of data being copied. Keep an eye on the logs being generated to know if everything worked as expected. 
