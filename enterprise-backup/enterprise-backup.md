# MYSQL ENTERPRISE BACKUP

## Introduction

MySQL Enterprise Backup is a backup utility for MySQL server. It is a multi-platform, high-performance tool, offering rich features like “hot” (online) backup, incremental and differential backup, selective backup and restore, support for direct cloud storage backup, backup encryption and compression, and many other valuable features.

**Note:** MySQL Enterprise Backup is available as part of MySQL Enterprise Edition distributions.


Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will:
* Execute a backup with MySQL Enterprise Backup
* Execute a restore with MySQL Enterprise Backup

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

## Task 1: Create a backup

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
    ```

2. Create a directory where store our backups

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mkdir -p /home/opc/backupdir/full</copy>
    ```

3. Using an administrative account for backup is not recommended for security, so we create now a dedicated user

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>CREATE USER 'mysqlbackup'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```

4. Backup user requires specific grants, sometimes related to instances with specific features enabled (e.g. InnoDB Cluster).
    Refer to the manual for details.
    For the purposes of this workshop these are the minimal

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>
    GRANT SELECT, BACKUP_ADMIN, RELOAD, PROCESS, SUPER, REPLICATION CLIENT ON *.* TO `mysqlbackup`@`%`;
    GRANT CREATE, INSERT, DROP, UPDATE ON mysql.backup_progress TO 'mysqlbackup'@'%';
    GRANT CREATE, INSERT, DROP, UPDATE, SELECT, ALTER ON mysql.backup_history TO 'mysqlbackup'@'%';
    </copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\q</copy>
    ```

5. Create now a new login path **mysqlbackup** 

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysql_config_editor set --login-path=mysqlbackup --user=mysqlbackup --host=127.0.0.1 -p</copy>
    ```

6. We are now ready to create a full backup 

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbackup --defaults-file=/etc/my.cnf --login-path=mysqlbackup --backup-dir=/home/opc/backupdir/full backup-and-apply-log</copy>
    ```

7. Have a look of the content of the backup folder

    a. Go to the backup directory
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <span style="color:green">shell></span> <copy>cd /home/opc/backupdir/full</copy>
    ```

    b. Check the content of "full" directory. You will see a copy of the datadir and backup metadata (including a copy of the backup log), and the my.cnf files created by MySQL Enterprise Backup
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <span style="color:green">shell></span> <copy>ls -l</copy>
    ```

    c. Check the content of backup "datadir" directory. You will see a copy of the original datadir, with renamed auto.cnf and mysqld-auto.cnf files
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <span style="color:green">shell></span> <copy>ls -l datadir/</copy>
    ```

    d. Check the content of backup "meta" directory. You will see the backup metadata, including the log
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <span style="color:green">shell></span> <copy>ls -l meta/</copy>
    ```

    e. Check the content of backup log.
    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <span style="color:green">shell></span> <copy>cat meta/MEB_*.log</copy>
    ```

## Task 2: Restore
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
    </span> <copy>sudo ls /var/lib/mysql/</copy>
    ```

3. Restore the backup. We use ***<code>sudo</code>*** because teh owner must be mysql (as opc user we have only read access)

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mysqlbackup --defaults-file=/etc/my.cnf --backup-dir=/home/opc/backupdir/full/ copy-back</copy>
    ```

    > NOTE: the "WARNING" is to rememebr that a restore on a different server than the original one may requires additional steps

5. Before start the instance, usually rename the files backup-auto.cnf and backup-mysqld-auto.cnf (but check if it apply to your use case)

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mv /var/lib/mysql/backup-auto.cnf /var/lib/mysql/auto.cnf</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mv /var/lib/mysql/backup-mysqld-auto.cnf /var/lib/mysql/mysqld-auto.cnf</copy>
    ```

6. We restored as root, so it's improtant to restore also the correct datadir ownership

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo chown -R mysql:mysql /var/lib/mysql/</copy>
    ```

7. We are now ready to restart the server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl start mysqld</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl status mysqld</copy>
    ```

7. Verify that data are restored

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost --table -e "SHOW DATABASES"</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost --table -e "SELECT * FROM employees.employees limit 10"</copy>
    ```

You may now **proceed to the next lab**

## Learn More
* https://dev.mysql.com/doc/mysql-enterprise-backup/8.4/en/mysqlbackup.tasks.html
* https://dev.mysql.com/doc/mysql-enterprise-backup/8.4/en/mysqlbackup.privileges.html


## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025
