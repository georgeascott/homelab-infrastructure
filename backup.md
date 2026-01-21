# BACKUP.md

## 1. Strategy Overview
This project follows a **3-2-1 Backup Strategy** to ensure data integrity and system availability for the NVR and Home Automation stack.

* **3 Copies of Data:** (1) Production/Live, (2) Local Archive, (3) Offsite Cloud.
* **2 Different Media:** Local SSD (Primary) and External HDD (Archive).
* **1 Offsite Copy:** Encrypted cloud storage.

## 2. Recovery Point Objective (RPO)
* **Configuration Files:** 24-hour RPO (Daily snapshots).
* **Databases (SQLite):** 24-hour RPO.
* **Media (Frigate Recordings):** 7-day RPO (Priority given to configuration over mass storage).

## 3. Pre-Migration SOP (Standard Operating Procedure)
When performing structural changes (e.g., moving directories from `~/` to `/opt/stacks`), the following steps are mandatory:

### Step 1: Services Shutdown
To prevent database corruption (SQLite), services must be fully stopped to release file locks.

```
docker compose -f /opt/stacks/frigate/docker-compose.yml down
```

### Step 2: Cold Archive Creation
Create a compressed, timestamped archive of the stack directory.
```
tar -cvzf backup_frigate_$(date +%F).tar.gz /opt/stacks/frigate
```

### Step 3: Checksum Verification
Verify the integrity of the backup file.
```
sha256sum backup_frigate_YYYY-MM-DD.tar.gz
```

## 4. Automation & Maintenance
Backups are managed via a custom bash script (`backup.sh`) triggered by a cron job.

## 5. Recovery Procedure
1. Navigate to the `/opt/stacks` directory.
2. Extract the archive: `tar -xvzf [BACKUP_FILE].tar.gz`
3. Validate the `docker-compose.yml` paths.
4. Relaunch: `docker compose up -d`
