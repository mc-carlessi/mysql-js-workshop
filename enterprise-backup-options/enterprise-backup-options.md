# MySQL Enterprise Backup

## Introduction
MySQL Enterprise Backup has multiple options, like incremental and/or differential backups, options for backup performances, and much more.
We explore here some.

Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will:
* Create a compressed backup
* Create a single image backup
* Create a differential backup

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

## Task 1: Create a compressed backup

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
    ```

2. Create a directory where store our compressed backup

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mkdir -p /home/opc/backupdir/compressed</copy>
    ```

3. Create a full and compressed backup 

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbackup --defaults-file=/etc/my.cnf --login-path=mysqlbackup --backup-dir=/home/opc/backupdir/compressed --compress backup-and-apply-log</copy>
    ```

4. Have a look of the content of the backup folder

    a. Go to the backup directory
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <span style="color:green">shell></span> <copy>cd /home/opc/backupdir/compressed</copy>
    ```

    b. Check the content. You will see a copy of the datadir and backup metadata (including a copy of the backup log)
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <span style="color:green">shell></span> <copy>ls -l</copy>
    ```

5. Compare the size of the previous full with the new one

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    </span> <copy>du -sh /home/opc/backupdir/*</copy>
    ```


## Task 2: Create a single image backup

1. Create a directory where store our compressed backup and one to store temporary files

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mkdir -p /home/opc/backupdir/image /home/opc/backupdir/tmp</copy>
    ```

2. Create a full backup stored in a single file 

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbackup --defaults-file=/etc/my.cnf --login-path=mysqlbackup --backup-dir=/home/opc/backupdir/tmp --backup-image=/home/opc/backupdir/image/mybck.mbi backup-to-image</copy>
    ```

3. Have a look of the content of the backup folder

    a. Go to the backup directory
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>cd /home/opc/backupdir/image</copy>
    ```

    b. Check the content. You will see a copy of the datadir and backup metadata (including a copy of the backup log)
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls -l</copy>
    ```

4. Verify the image backup file

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbackup --backup-image=/home/opc/backupdir/image/mybck.mbi validate</copy>
    ```

5. List the content of the image backup. Please note that the image contains also backup metatadata

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbackup --backup-image=/home/opc/backupdir/image/mybck.mbi list-image</copy>
    ```


## Task 3: Create a differential backup
1. Create a directory where store our differential backup

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mkdir -p /home/opc/backupdir/differential</copy>
    ```

2. Create a new table with some data in our instance

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost -e "CREATE DATABASE test"</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost -e "CREATE TABLE test.mytab (id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT, val VARCHAR(10), PRIMARY KEY(id))"</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost -e "INSERT INTO test.mytab(val) VALUES('a'),('b'),('c')"</copy>
    ```

3. Empty temporary directory

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>rm -rf /home/opc/backupdir/tmp/*</copy>
    ```

4. Create an incremental backup based on previous full  full and compressed backup 

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbackup --defaults-file=/etc/my.cnf --login-path=mysqlbackup --incremental --incremental-base=dir:/home/opc/backupdir/full/ --incremental-backup-dir=/home/opc/backupdir/differential backup</copy>
    ```

5. Have a look of the content of the backup folder

    a. Go to the backup directory
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>cd /home/opc/backupdir/differential</copy>
    ```

    b. Check the content. You will see a copy of the datadir and backup metadata (including a copy of the backup log)
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls -l</copy>
    ```

You may now **proceed to the next lab**

## Learn More
* https://dev.mysql.com/doc/mysql-enterprise-backup/8.4/en/mysqlbackup.tasks.html
* https://dev.mysql.com/doc/mysql-enterprise-backup/8.4/en/mysqlbackup.privileges.html


## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025
