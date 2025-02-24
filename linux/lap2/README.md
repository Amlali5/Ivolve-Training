
# MySQL Backup Automation

This project sets up MySQL server backup automation on a Linux system using a Bash script and a cron job.

## Steps

### 1. Install MySQL Server
1. Use the following command to install MySQL server:
   ```bash
   sudo dnf -y install mysql-server
   ```
2. Enable and start the MySQL service:
   ```bash
   sudo systemctl enable mysqld
   sudo systemctl start mysqld
   sudo systemctl status mysqld
   ```

### 2. Create the Backup Script
1. Navigate to the `/usr/local/bin/` directory.
2. Create a backup script called `mysql_backup.sh`:
   ```bash
   sudo vim /usr/local/bin/mysql_backup.sh
   ```
3. Add the following content to the script:
   ```bash
   #!/bin/bash
   BACKUP_DIR="/backup/mysql"
   TIMESTAMP=$(date +%F-%H-%M-%S)
   BACKUP_FILE="mysql-backup-$TIMESTAMP.sql"

   # Ensure the backup directory exists
   mkdir -p "$BACKUP_DIR"

   # Dump all MySQL databases
   mysqldump -u root --password=YOUR_PASSWORD --all-databases > "$BACKUP_DIR/$BACKUP_FILE"

   # Compress the backup file
   gzip "$BACKUP_DIR/$BACKUP_FILE"

   echo "MySQL backup completed: $BACKUP_DIR/$BACKUP_FILE.gz"
   ```
4. Replace `YOUR_PASSWORD` with the root password for your MySQL instance.
5. Save and exit the file.

### 3. Make the Script Executable
Set executable permissions for the script:
```bash
sudo chmod +x /usr/local/bin/mysql_backup.sh
```

### 4. Test the Backup Script
Run the script manually to ensure it works:
```bash
sudo /usr/local/bin/mysql_backup.sh
```

### 5. Verify the Backup
Check the `/backup/mysql/` directory for the compressed backup file:
```bash
ls -l /backup/mysql
```

### 6. Automate Backups with Cron
1. Open the crontab for editing:
   ```bash
   crontab -e
   ```
2. Add the following line to schedule the script to run every Sunday at 5:00 PM:
   ```bash
   0 17 * * 0 /usr/local/bin/mysql_backup.sh
   ```
3. Save and exit the crontab editor.

### 7. List Cron Jobs
Verify that the cron job is installed:
```bash
crontab -l
```
