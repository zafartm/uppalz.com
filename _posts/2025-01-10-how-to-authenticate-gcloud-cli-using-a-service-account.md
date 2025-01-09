---
layout: post
title:  "How to deploy a multi container application to a docker server"
---

# How to authenticate gcloud-cli using a service account

This applies to such use cases where using a personal account is not an option. For example to run a cronjob in a remote server.

1. Create a service account and download its key file. Assume `my-account@my-project.iam.gserviceaccount.com` is the service account email and `my-account-key.json` is the key file for this example.
2. Assign appropriate roles to the account.
3. Activate the service account in gcloud config; 
   ```commandline
   gcloud auth activate-service-account --key-file=my-account-key.json
   ```
4. Select the service account;
   ```commandline
   gcloud config set account my-account@my-project.iam.gserviceaccount.com
   ```
