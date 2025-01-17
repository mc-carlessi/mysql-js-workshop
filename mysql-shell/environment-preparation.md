# INSTALL - VERIFY MYSQL ENTERPRISE EDITION  

## Introduction

Goal:
    Verify the new MySQL Installation on Linux and import test databases

Objectives:

- understand better how MySQL connection works
- have a look on useful statements

Estimated Time: -- 10 minutes

### Objectives

In this lab, you will:

- Connect to Ports
- Learn Useful SQL Statements

### Prerequisites

This lab assumes you have:

- All previous labs successfully completed

### Lab standard

Pay attention to the prompt, to know where execute the commands 
* ![green-dot](./images/green-square.jpg) shell>  
  The command must be executed in the Operating System shell
* ![blue-dot](./images/blue-square.jpg) mysql>  
  The command must be executed in a client like MySQL, MySQL Shell or similar tool
* ![yellow-dot](./images/yellow-square.jpg) mysqlsh>  
  The command must be executed in MySQL shell

## Task 1: MySQL Connection

Please note that password can be saved in an obfuscated file, to be reused, using the utility mysql_config_editor.  
This approach let you configure more secure and flexible scripts.

1. Set the login path **local_admin** to be used for easier connections 

 **![green-dot](./images/green-square.jpg) shell>**
    ```
    <copy>mysql_config_editor set --login-path=local_admin --user=admin --host=127.0.0.1 -p</copy>
    ```

2. Test the connection with mysql and mysqlsh clients

 **![green-dot](./images/green-square.jpg) shell>**
    ```
    <copy>mysql --login-path=local_admin</copy>
    ```
 **![blue-dot](./images/blue-square.jpg) mysql>** 
    ```
    <copy>exit</copy>
    ```

 **![green-dot](./images/green-square.jpg) shell>**
    ```
    <copy>mysqlsh --login-path=local_admin</copy>
    ```
 **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <copy>\quit</copy>
    ```

2. List existing settings, please note that password are obfuscated

 **![green-dot](./images/green-square.jpg) shell>**
    ```
    <copy>mysql_config_editor print --all</copy>
    ```

## Task 2: Learn Useful SQL Statements

1. Connect to your instance

    **![green-dot](./images/green-square.jpg) shell>**
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

2. You can check the values of a specific variable, like the version of your server to know if it's updated to last release or not  
    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <copy>SHOW VARIABLES LIKE "%version%";</copy>
    ```

3. InnoDB provides the best Storage Engine in the general use case. You can check if there are non-InnoDB tables (of course, excluding system schemas)
    ```
    <copy>SELECT table_schema, table_name, engine FROM INFORMATION_SCHEMA.TABLES where engine <> 'InnoDB' and table_schema not in ('mysql','information_schema', 'sys', 'performance_schema', 'mysql_innodb_cluster_metadata');</copy>
    ```

4. You can check the amount of data inside all the databases  
    ```
    <copy>SELECT table_schema AS 'Schema', SUM( data_length ) / 1024 / 1024 AS 'Data MB', SUM( index_length ) / 1024 / 1024 AS 'Index MB', SUM( data_length + index_length ) / 1024 / 1024 AS 'Sum' FROM information_schema.tables GROUP BY table_schema ;</copy>
    ```

5. You can check the amount of data inside all tables in a specific database ('employees' in the example)  
    ```
    <copy>SELECT table_schema AS 'Schema', table_name, SUM( data_length ) / 1024 / 1024 AS 'Data MB', SUM( index_length ) / 1024 / 1024 AS 'Index MB', SUM( data_length + index_length ) / 1024 / 1024 AS 'Sum' FROM information_schema.tables WHERE table_schema='employees' GROUP BY table_name;</copy>
    ```

6. You can check the size of tablespaces files for a specific database ('employees' in the example)  
    ```
    <copy>SELECT name, space AS 'tablespace id', allocated_size /1024 /1024 AS 'size (MB)', encryption FROM information_schema.innodb_tablespaces WHERE name LIKE 'employees/%';</copy>
    ```

7. The “\G” is like “;” with a different way to show results

  **![yellow-dot](./images/yellow-square.jpg) mysqlsh>** 
    ```
    <copy>SHOW GLOBAL VARIABLES LIKE 'version%';</copy>
    ```
    ```
    <copy>SHOW GLOBAL VARIABLES LIKE 'version%'\G</copy>
    ```

8. Show connections

  **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>SHOW FULL PROCESSLIST;</copy>
    ```

9. You can now exit
    ```
    <copy>\q</copy>
    ```

## Learn More

* [MySQL Tutorial](https://dev.mysql.com/doc/en/tutorial.html)

## Acknowledgements

- **Author** - Dale Dasker, MySQL Solution Engineering
- **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
