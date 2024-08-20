---
layout: post
title: "How to move data from one mongo server to the other using mongodump and mongorestore"
---

# How to move data from one mongo server to the other using mongodump and mongorestore

### 1. Get into the source server

Depending upon the env, you may need to use one of the `ssh`, `docker exec`, `docker compose exec` or `kubectl exec` to get into the mongo server.

### 2. Dump the source databases. One db at a time

```shell
mongodump --db=<db_name> --gzip --archive=/tmp/db_name_dump.gz
```

### 3. Move dump file from source server to the target server

Depending upon the env, you may need to use one of the `scp`, `docker cp`, `docker compose cp` or `kubectl cp` commands to
copy `/tmp/*_dump.gz` files from source server to the target.

### 4. Get into the target server

`ssh` | `docker exec` | `docker compose exec` | `kubectl exec`

### 5. Do a dry run before restoring

```shell
mongorestore --gzip --archive=/tmp/db_name_dump.gz --dryRun --verbose
```

Observe the screen output. Proceed only if you are confident that you are not overwriting any wrong db or collection

### 6. To change the database name during restore

By default, mongorestore uses the original database name. If there is any need to change the database name during restore,
this command may be used.

```shell
mongorestore --gzip --archive=/tmp/db_name_dump.gz --nsFrom="<old_db_name>.*" --nsTo="<new_db_name>.*" --dryRun --verbose
```

### 7. Do the actual restore

Make sure the command is structured correctly by doing a dry run first. Once you are confident, omit the `--dryRun` option from the command and viola!

```shell
mongorestore --gzip --archive=/tmp/db_name_dump.gz
```
