# MySQL Logical Backup

## Introduction
The mysqldump client utility performs logical backups, producing a set of SQL statements that can be executed to reproduce the original database object definitions and table data. It dumps one or more MySQL databases for backup or transfer to another SQL server. 

In this lab you will see how mysqldump works.

Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will:
* execute backups with mysqldump 
* execute restore from a backup created with mysqldump 

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


## Task 1: mysqldump - backup

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
    ```

2. Create the export folder

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mkdir -p /home/opc/exports</copy>
    ```

3. Export all the data with mysqldump. We use here the ***login_path*** to authenticate, easier than specify ***<code>"-u <user> -h <host> -p"</code>***

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqldump --login-path=local_admin --single-transaction --events --routines --flush-logs --all-databases > /home/opc/exports/full.sql</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls -l /home/opc/exports/full.sql</copy>
    ```

3. View the content of the file full.sql. 
   Please note that "database:" in third line is empty, and first database exported is "mysql"
   > Note:use CTRL + x to close nano editor without changes

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>nano /home/opc/exports/full.sql</copy>
    ```

    ```sql
    -- MySQL dump 10.13  Distrib 8.4.3, for Win64 (x86_64)
    --
    -- Host: localhost    Database: 
    -- ------------------------------------------------------
    -- Server version	8.4.3-commercial

    /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
    /*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
    /*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
    /*!50503 SET NAMES utf8mb4 */;
    /*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
    /*!40103 SET TIME_ZONE='+00:00' */;
    /*!50606 SET @OLD_INNODB_STATS_AUTO_RECALC=@@INNODB_STATS_AUTO_RECALC */;
    /*!50606 SET GLOBAL INNODB_STATS_AUTO_RECALC=OFF */;
    /*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
    /*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
    /*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
    /*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

    --
    -- Current Database: `mysql`
    --

    CREATE DATABASE /*!32312 IF NOT EXISTS*/ `mysql` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;
    ```

4. Export a specific database (employees)

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqldump --login-path=local_admin --single-transaction --set-gtid-purged=OFF --databases employees > /home/opc/exports/employees_only.sql</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls -l /home/opc/exports/employees_only.sql</copy>
    ```

5. View  the content of the file employees.sql.
   Please note that "database:" in third line is "employees" 
   > Note:use CTRL + x to close nano editor without changes

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>nano /home/opc/exports/full.sql</copy>
    ```

    ```sql
    -- MySQL dump 10.13  Distrib 8.4.3, for Win64 (x86_64)
    --
    -- Host: localhost    Database: employees
    -- ------------------------------------------------------
    -- Server version	8.4.3-commercial

    /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
    /*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
    /*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
    /*!50503 SET NAMES utf8mb4 */;
    /*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
    /*!40103 SET TIME_ZONE='+00:00' */;
    /*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
    /*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
    /*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
    /*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

    --
    -- Current Database: `employees`
    --

    CREATE DATABASE /*!32312 IF NOT EXISTS*/ `employees` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;
    ```

## Task 2: mysqldump restore 

1. Drop employees database, using the mysql client

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysql --login-path=local_admin</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysql>**
    ```
    <copy>SHOW DATABASES;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysql>**
    ```
    <copy>DROP DATABASE employees;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysql>**
    ```
    <copy>SHOW DATABASES;</copy>
    ```

2. Import the dropped employees database.

    **![orange-dot](./images/orange-square.jpg) mysql>**
    ```
    <copy>SOURCE /home/opc/exports/employees_only.sql</copy>
    ```

3. Check that content is restored
    **![orange-dot](./images/orange-square.jpg) mysql>**
    ```
    <copy>SHOW DATABASES;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysql>**
    ```
    <copy>SHOW TABLES IN employees;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysql>**
    ```
    <copy>SELECT * FROM employees.employees limit 10;</copy>
    ```

7. Exit from mysql client

    **![orange-dot](./images/orange-square.jpg) mysql>**
    ```
    <copy>exit</copy>
    ```
    
## Learn More
* [MySQL Dump manual page](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html)

## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025

