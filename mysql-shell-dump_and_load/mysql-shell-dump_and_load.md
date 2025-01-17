# MySQL Logical Backup

## Introduction
MySQL Shell's can be used to efficiently create a logical export of the instance
- instance dump utility util.dumpInstance(), to export the full instance content
- schema dump utility util.dumpSchemas(), to export all schemas or a selected schema
- table dump utility util.dumpTables(), to export a selection of tables or views from a schema

MySQL Shell's dump loading utility util.loadDump() supports the import into a MySQL Server instance of schemas or tables dumped using MySQL Shell's dump utilities.
The dump loading utility provides data streaming from remote storage, parallel loading of tables or table chunks, progress state tracking, resume and reset capability, and the option of concurrent loading while the dump is still taking place.

In this lab you will see how Mysql Shell dump & load works, and compare performances with mysqldump.

Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will:
* Introduction to MySQL Shell
* execute a backup with MySQL Shell dump utility
* execute a schema restore with MySQL Shell load utility 
* compare execution time between mysqldump and MySQL Shell

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



## Task 1: MySQL Shell overview

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
    ```
   
2. Connect with MySQL Shell to your instance

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

## Task 2: MySQL Shell dump
1. Switch to javascript command mode to check the export the employee database.  
    To ***just check*** if export parameters are correct, without execute the export, we can execute the command in dry-run mode

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>\js</copy>
    ```

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.dumpSchemas(['employees'],'/home/opc/exports/employees',{dryRun:true})</copy>
    ```
2. If there are no errors, execute the export without dryRun.  

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.dumpSchemas(['employees'],'/home/opc/exports/employees')</copy>
    ```

3. Check the content of the directory /home/opc/exports/employees

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>\q</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls -l /home/opc/exports/employees</copy>
    ```

## Task 3: MySQL Shell load

1. Reconnect mysqlsh and drop employees database

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysql>**  
    ```
    <copy>DROP DATABASE employees;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysql>**  
    ```
    <copy>SHOW DATABASES;</copy>
    ```

2. To load files, MySQL Shell requires local_infile variable enable (for security reasons is disabled by defautl)

    **![orange-dot](./images/orange-square.jpg) mysql>**  
    ```
    <copy>SET PERSIST local_infile=ON;</copy>
    ```

3. Switch to javascript command mode and test if parameters are correct using dryRun mode

    **![orange-dot](./images/orange-square.jpg) mysql>**  
    ```
    <copy> \js</copy>
    ```

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.loadDump('/home/opc/exports/employees',{dryRun:true})</copy>
    ```

4. If there are no errors, load the employees database.  

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.loadDump('/home/opc/exports/employees')</copy>
    ```

5. Check that the database employees is imported. It's interesting to note that the ***"<code>\sql</code>"*** prefix let us execute SQL statements without swith to SQL mode

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>\sql SHOW DATABASES;</copy>
    ```

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>\q</copy>
    ```

## Task 4: MySQL Shell command line and comparison to mysqldump

1. MySQL Shell can be executed also at command line (useful for scripting) with "<code>mysqlsh [options] -- shell_object
object_method [method_arguments]</code>"

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1 -- util dumpSchemas --help</copy>
    ```

2. We can execute the dump and note the "Total duration" time

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1 -- util dumpSchemas "['employees']" --outputUrl='/home/opc/exports/employees_shell_time'</copy>
    ```

3. We can also calculate the time with mysqldump using the <code>time</code> utility provided by linux, that tell us how much time is required to execute a specific command, and compare with previous timing

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>time mysqldump --login-path=local_admin --single-transaction --set-gtid-purged=OFF --databases employees > /home/opc/exports/employees_mysqldump_time.sql</copy>
    ```

4. We compare now the import timing. First drop the employees database  

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1 -e "DROP DATABASE employees"</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1 --table -e "SHOW DATABASES"</copy>
    ```

5. Let's now compare execution time of the import. We start with MySQL Shell load and note the "Total duration" time

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost -- util loadDump '/home/opc/exports/employees_shell_time'</copy>
    ```

6. Drop the employees database  

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1 -e "DROP DATABASE employees"</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1 --table -e "SHOW DATABASES"</copy>
    ```

7. Execute mysql import from mysqldump.  
    Please **compare the mysql import time** with the MysQL load.

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>time mysql --login-path=local_admin < /home/opc/exports/employees_mysqldump_time.sql</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysql --login-path=local_admin -e "show databases"</copy>
    ```

You may now **proceed to the next lab**

## Learn More
* [mysqldump manual page](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html)
* [MySQL Shell dump utilities](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-dump-instance-schema.html)
* [MySQL Shell load utility](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-load-dump.html)
* [MySQL Shell at command line](https://dev.mysql.com/doc/mysql-shell/8.0/en/command-line-integration-overview.html)


## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025
