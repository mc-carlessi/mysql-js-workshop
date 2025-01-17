# INSTALL - MYSQL SHELL CLIENT AND MYSQL_CONFIG_EDITOR  

## Introduction
MySQL Shell is an advanced client and code editor for MySQL. In addition to the provided SQL functionality, similar to mysql, MySQL Shell provides scripting capabilities for JavaScript and Python and includes APIs for working with MySQL.
In this lab we learn more about it.

The mysql_config_editor utility enables you to store authentication credentials in an obfuscated login path file named .mylogin.cnf.
This utility let us increase security and manageability of scripting (for example for backup purposes), and works with all mysql command line utilities.

Goal:
- Practice with MySQL Shell
- practice with mysql_config_editor

Estimated Time:  10 minutes

### Objectives

In this lab, you will:

- Understand how MySQL Shell works

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
* ![blue-dot](./images/blue-square.jpg) mysqlsh>  
  The command must be executed in MySQL shell python command mode

## Task 1: MySQL Shell introduction

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
    ```

2. MySQL shell offers various utilities, so it can be started without a connection to a MySQL server instance

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    </span> <copy>mysqlsh</copy>
    ```

3. It provides command auto-completion feature. To test it, type \h and press TAB twice

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\h</copy> [TAB][TAB]
    ```

4. Check the available options. Add the letter “e” to “\h” and press TAB again to see that the command will automatically complete for you. Press enter and explore the help menu

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\he</copy> [TAB]
    ```

5. MySQL Shell offers various configuration options. List them

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\option -l</copy>
    ```

6. Activate the command history autosave in MySQL shell  
    MySQL Shell comes with the option of automatically saving the history of command disabled by default. Therefore we need to check and to activate it.

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    * Activate autosave history
    ```
    <copy>\option --persist history.autoSave=1</copy>
    ```


## Task 2: MySQL as advanced SQL Client

1. MySQL Shell can be used as SQL client.  
   Connect to the newly installed mysql-advanced instance. Enter the password when requested and press enter. 
    
   When the prompt asks to save the password choose No and press Enter.

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\c admin@127.0.0.1</copy>
    ```
2. Now you can submit SQL commands

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>show databases;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>show tables in employees;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>SELECT * FROM employees.employees LIMIT 10;</copy>
    ```

3. MySQL Shell has also reporting features.
   As an example, we can check the actual sessions with a query

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>select * from performance_schema.processlist;</copy>
    ```

3. instead of continuously manually repeat the command from history, we can use the reporting feature that continuously repeat a query (default interval refresh time: 5 seconds).
    To exit from reporting press CTRL+C

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\watch query --interval=2 select now(), processlist.* from performance_schema.processlist;</copy>
    ```

4. MySQL Shell has also a command mode, where is possible to create and manage clusters, execute dump and loads, and much more. Command mode can be in python (\\py) or javascript (\\js).
    Please note how the prompt change accordingly to our selection

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\py</copy>
    ```

    **![blue-dot](./images/blue-square.jpg) mysqlsh>**
    ```
    <copy>\js</copy>
    ```

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**
    ```
    <copy>\sql</copy>
    ```

5. To exit from MySQL Shell type “\q” or “\exit”

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\q</copy>
    ```

## Task 3: MySQL Connection with mysql\_config\_editor

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
    
    **![orange-dot](./images/orange-square.jpg) mysql>**
    ```
    <copy>\quit</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh --login-path=local_admin</copy>
    ```
    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\quit</copy>
    ```

2. List existing settings, please note that password are obfuscated

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysql_config_editor print --all</copy>
    ```

## Task 3: Useful SQL Statements

1. Connect to your instance

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

2. You can check the values of a specific variable, like the version of your server to know if it's updated to last release or not  
    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>SHOW VARIABLES LIKE "%version%";</copy>
    ```

3. InnoDB provides the best Storage Engine in the general use case. You can check if all are tables in a different format (of course, excluding system schemas)
    
    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>SELECT table_schema table_name, engine FROM INFORMATION_SCHEMA.TABLES where engine <> 'InnoDB' and table_schema not in ('mysql','information_schema', 'sys', 'performance_schema', 'mysql_innodb_cluster_metadata');</copy>
    ```

4. Check the amount of data inside all the databases  

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>SELECT table_schema AS 'Schema', SUM( data_length ) / 1024 / 1024 AS 'Data MB', SUM( index_length ) / 1024 / 1024 AS 'Index MB', SUM( data_length + index_length ) / 1024 / 1024 AS 'Sum' FROM information_schema.tables GROUP BY table_schema ;</copy>
    ```

5. Check the amount of data inside all tables in a specific database ('employees' in the example)  

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>SELECT table_schema AS 'Schema', table_name, SUM( data_length ) / 1024 / 1024 AS 'Data MB', SUM( index_length ) / 1024 / 1024 AS 'Index MB', SUM( data_length + index_length ) / 1024 / 1024 AS 'Sum' FROM information_schema.tables WHERE table_schema='employees' GROUP BY table_name;</copy>
    ```

6. Check the size of tablespaces files for a specific database ('employees' in the example)  

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>SELECT name, space AS 'tablespace id', allocated_size /1024 /1024 AS 'size (MB)', encryption FROM information_schema.innodb_tablespaces WHERE name LIKE 'employees/%';</copy>
    ```

7. Retrieve the datadir position

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>SELECT * FROM performance_schema.global_variables WHERE VARIABLE_NAME LIKE 'datadir';</copy>
    ```

8. Retrieve the my.cnf/my.ini configuration files position

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>SELECT DISTINCT (VARIABLE_PATH) FROM performance_schema.variables_info;</copy>
    ```


9. You can now exit from MySQL Shell and proceed to next lab

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\q</copy>
    ```

## Learn More

* [MySQL Shell manual](https://dev.mysql.com/doc/mysql-shell/8.4/en/)
* [mysql\_config\_editor manual page](https://dev.mysql.com/doc/refman/8.4/en/mysql-config-editor.html)

## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025
