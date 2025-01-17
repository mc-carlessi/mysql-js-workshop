# MySQL Enterprise Backup

## Introduction
Point-in-time recovery (PiTR) refers to recovery of data changes up to a given point in time. Typically, this type of recovery is performed after restoring a full backup that brings the server to its state as of the time the backup was made. 
The full backup can be made in several ways (import from a mysqldump file, MySQL Shell load, MySQL Enterprise BAckup restore).
Binary logs are used to achieve a PiTR binary and the utility is mysqlbinlog.
To speed up the process is also possible to use replica concepts, as explained in [KB 2009693.1](https://support.oracle.com/epmos/faces/DocumentDisplay?id=2009693.1) and [KB 2277457.1](https://support.oracle.com/epmos/faces/DocumentDisplay?id=2277457.1), both available on support website [https://support.oracle.com](https://support.oracle.com).

Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will:
* Create a full backup
* Apply some changes
* Restore the backup with the full backup and the binary logs

### Prerequisites

This lab assumes you have:
- All previous labs successfully completed

### Lab standard

Pay attention to the prompt, to know where execute the commands 
* ![green-dot](./images/green-square.jpg) shell>  
  The command must be executed in the Operating System shell
* ![orange-dot](./images/orange-square.jpg) mysql>  
  The command is SQL and must be executed in a client like MySQL, MySQL Shell or similar tool
* ![yellow-dot](./images/yellow-square.jpg) mysqlsh>  
  The command must be executed in MySQL shell javascript command mode


## Task 1: prepare the instance and create the full backup

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
    ```

2. MySQL binary logs are  enabled by default in MySQL 8. But it's recommended to use GTID, not enabled by default.
    Let's enable now the binary logs

    a. Edit my.cnf

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo nano /etc/my.cnf</copy>
    ```
    b. Add these lines to my.cnf and save the file (CTRL o + CTRL X)

    ```
    gtid_mode=on
    enforce_gtid_consistency=on
    ```

    c. Restart the instance adn reviry that it's started

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl restart mysqld</copy>
    ```
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl status mysqld</copy>
    ```

3. Create a directory where store our backups

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mkdir -p /home/opc/backupdir/full_pitr</copy>
    ```

4. We are now ready to create a full backup 
    Note: we are creating a new backup, and not using one previously created, because we just enabled GTID mode.

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbackup --defaults-file=/etc/my.cnf --login-path=mysqlbackup --backup-dir=/home/opc/backupdir/full_pitr backup-and-apply-log</copy>
    ```

## Task 2: Execute some commands to test PiTR

1. Connect with MySQL Shell 

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

2. Create a table and add some records 

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>CREATE DATABASE test;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>USE test;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>CREATE TABLE pets (id BIGINT UNSIGNED NOT NULL, pet varchar(50), primary key(id));</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>INSERT INTO pets VALUES(1,'dog'),(2,'cat'),(3,'bunny');</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>select * from pets;</copy>
    ```

## Task 3: Save binary logs

1. Check binary logs 

    a. List existing binary logs

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SHOW BINARY LOGS;</copy>
    ```

    b. Force binary logs rotation

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SHOW BINARY LOGS;</copy>
    ```

    c. List existing binary logs

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SHOW BINARY LOGS;</copy>
    ```

    c. Check where binary logs are saved. From this qyery you can see the path ('/var/lib/mysql/') and the basename file ('binlog')

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SELECT @@log_bin_basename;</copy>
    ```

    D. Exit from MySQL Shell
    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>\q</copy>
    ```

2. Create a directory where store our binary logs

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mkdir /home/opc/backupdir/binlogs</copy>
    ```

3. The file backup_variables.txt created by MySQL Enterprise Backup list backup positions.
    We use it to save the binary logs created after the backup

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>cat /home/opc/backupdir/full_pitr/meta/backup_variables.txt</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>grep binlog_position /home/opc/backupdir/full_pitr/meta/backup_variables.txt</copy>
    ```

4. To simplify next commands reading let's set some variables

    a. Set basedir
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>basedir='/var/lib/mysql/'</copy>
    ```

    b. Set backup_variables.txt file
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>variables_file='/home/opc/backupdir/full_pitr/meta/backup_variables.txt'</copy>
    ```

    c. Set the directory where backup binary logs
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>destination_backup='/home/opc/backupdir/binlogs/'</copy>
    ```

5. Copy the binlog just retrieved in the binlog backup folder.
    You can do it manually or use the command below.

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>cp "$datadir"/"$( grep binlog_position $variables_file | sed -e 's/^.*=//g' -e 's/\:.*$//g' )" $destination_backup</copy>
    ```

6. Copy all the binlogs created after that one 
    You can do it manually or use the command below.

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>echo $( ls $datadir/binlog.[0-9]* ) | sed -e 's/^.*'$( grep binlog_position $variables_file | cut -d\= -f 2 | cut -d\: -f 1 )'//g#'</copy>
    ```

7. List the binary logs in our baclup directory

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls /home/opc/backupdir/binlogs</copy>
    ```

## Task 3: Destroy the instance and restore the backup
1.  Stop the server, to exclude unexpected behaviors from running processes

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl stop mysqld</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl status mysqld</copy>
    ```

2. Empty the datadir (if you don't know where is the datadir, read the section [Useful SQL Statements](../mysql-shell/mysql-shell.md#task-3-useful-sql-statements))

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    </span> <copy>ls /var/lib/mysql/</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    </span> <copy>sudo rm -rf /var/lib/mysql/*</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    </span> <copy>ls /var/lib/mysql/</copy>
    ```

3. Restore the backup. We use ***<code>sudo</code>*** because of ownership

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mysqlbackup --defaults-file=/etc/my.cnf --backup-dir=/home/opc/backupdir/full_pitr/ copy-back</copy>
    ```

5. Before start the instance, if needed, rename the files backup-auto.cnf and backup-mysqld-auto.cnf

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mv /var/lib/mysql/backup-auto.cnf /var/lib/mysql/auto.cnf</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mv /var/lib/mysql/backup-mysqld-auto.cnf /var/lib/mysql/mysqld-auto.cnf</copy>
    ```

6. Set the correct ownership

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo chown -R mysql:mysql /var/lib/mysql/</copy>
    ```

7. Start the server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl start mysqld</copy>
    ```

7. Verify that data are restored. You will see that the new database test is not available (backup was taken before it's creation)

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost --table -e "SHOW DATABASES"</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost --table -e "SELECT * FROM employees.employees limit 10"</copy>
    ```

## Task 4: Execute the PiTR with binary logs

1. Move to the saved binary logs directory

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>cd /home/opc/backupdir/binlogs</copy>
    ```

2. List binary logs by name

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls -l</copy>
    ```

3. Inspect the content of the last binary log using mysqlbinlog utility (you can manually specify the name, or use the shell commands to retrieve the file for you).
    This command displays row events encoded as base-64 strings (difficult to read for humans)  

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbinlog $(ls | tail -1 )</copy>
    ```

4. Inspect the content of the last binary log in a human readable form.
    See ![here](https://dev.mysql.com/doc/refman/8.4/en/mysqlbinlog-row-events.html) for mysqlbinlog output display options

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbinlog --base64-output=DECODE-ROWS --verbose $(ls | tail -1 )</copy>
    ```

5. Retrieve the restored point of the backup in the binary logs from the backup_variables.txt file created during the restore process by MySQL Enterprise Backup.
    To simplify our job, let's assign the value to a variable

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>grep binlog_position /var/lib/mysql/backup_variables.txt</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>pos=$( grep binlog_position $variables_file | cut -d\= -f 2 | awk -F'[.]' 'NF>1 {print $NF}' | cut -d\: -f 2 )</copy>
    ```

6. Start the restore from that position (for the PiTR use mysql client and not MySQL Shell)

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbinlog --start-position=$pos $destination_backup/*.[0-9]* | mysql --login-path=local_admin</copy>
    ```

7. Verify that data are restored

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost --table -e "SHOW DATABASES"</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost --table -e "SELECT * FROM test.pets"</copy>
    ```

This **ends our workshop**

## Learn More
* ![Point in Time Recovery](https://dev.mysql.com/doc/refman/8.4/en/point-in-time-recovery.html)
* ![mysqlbinlog utility](https://dev.mysql.com/doc/refman/8.4/en/mysqlbinlog.html)
* ![MySQL Enterprise Backup Point in Time Recovery](https://dev.mysql.com/doc/mysql-enterprise-backup/8.4/en/advanced.point.html)

## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025
