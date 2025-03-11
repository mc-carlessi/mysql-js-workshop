# MYSQL JAVASCRIPT - INTRODUCTION

## Introduction

The Multilingual Engine component (MLE) provides support for languages other than SQL in MySQL stored procedures and functions. The MLE Component is available as part of MySQL Enterprise Edition.

With the MLE component in MySQL 9.2, you can create and execute MySQL stored programs written in JavaScript.

The MLE component uses graalvm to provide native support for JavaScript as a stored procedure language.  
Javascript enables more complex logic directly in the database and enhanced performance over traditional interpreted approaches.

> Note  
There are various limitations to Javascript implementation, for the full list [check the manual](https://dev.mysql.com/doc/refman/9.2/en/srjs-limitations.html).  
  * The MLE component uses a single-threaded execution model (one thread per query). This means that all asynchronous features like the JavaScript Promise object and async functions are simulated and can exhibit non-deterministic behavior.
  * As with SQL stored routines, JavaScript stored routines with a variable number of arguments are not supported. But JavaScript functions within routines can have a variable number of arguments.
  * The recursion depth is limited to 1000

> About the conversion between MySQL data types and Javascript data types, check the manual.  
Here some important notes
  * MySQL **DECIMAL** is not supported in Javascript
  * Javascript **Infinity**, **NaN**, or **Symbol** are not supported in MySQL  


Estimated Time: 15 minutes

### Objectives

In this lab, you will:

* Install MySQL Enterprise Edition
* Install MySQL Shell 
* Import a sample database


### Prerequisites

This lab assumes you have:
* A working Oracle Linux machine
* The MySQL Enterprise rpms for Oracle Linux inside /workshop directory
* The employees sample database ([sample databases are downloadable from dev.mysql.com](https://dev.mysql.com/doc/index-other.html))

### Lab standard

Pay attention to the prompt, to know where execute the commands 
* ![green-dot](./images/green-square.jpg) shell>  
  The command must be executed in the Operating System shell
* ![orange-dot](./images/blue-square.jpg) mysql>  
  The command is SQL and must be executed in a client like MySQL, MySQL Shell or similar tool
* ![yellow-dot](./images/yellow-square.jpg) mysqlsh>  
  The command must be executed in MySQL shell javascript command mode
  
## Task 1: Install and configure MLE component

1. If not already connected, connect to your mysql server

  **![green-dot](./images/green-square.jpg) shell>**  
  ```shell
  <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
  ```

2. Now connect yo your MySQL instance

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>mysqlsh admin@127.0.0.1</copy>
  ```

3. Let's increase the error log verbosity to see more information 

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SET PERSIST log_error_verbosity=3;</copy>
  ```

4. MLE component installation is standard.

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>INSTALL COMPONENT 'file://component_mle';</copy>
  ```

5. We can verify its installation listing the loaded components. Search the line with 'component_mle'

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT * FROM mysql.component;</copy>
  ```

  **OUTPUT:** 
  ```
  +--------------+--------------------+------------------------------------+
  | component_id | component_group_id | component_urn                      |
  +--------------+--------------------+------------------------------------+
  |            1 |                  1 | file://component_validate_password |
  |            2 |                  2 | file://component_mle               |
  +--------------+--------------------+------------------------------------+
  ```

6. We can also check errors and warnings during the load, with the usual error log  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT * FROM performance_schema.error_log WHERE SUBSYSTEM='MLE';</copy>
  ```

  **OUTPUT:** 
  ```
  +----------------------------+-----------+------+------------+-----------+---------------------------------+
  | LOGGED                     | THREAD_ID | PRIO | ERROR_CODE | SUBSYSTEM | DATA                            |
  +----------------------------+-----------+------+------------+-----------+---------------------------------+
  | 2025-03-05 09:52:02.074877 |         9 | Note | MY-015000  | MLE       | Component starting...           |
  | 2025-03-05 09:52:02.080259 |         9 | Note | MY-015000  | MLE       | Component state: INACTIVE       |
  +----------------------------+-----------+------+------------+-----------+---------------------------------+
  ```

7. Status of the component can be retrieved with a show status statement. Component was just started, but never used, so not loaded.  
  For this reason majority of the variables are just initialized.

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SHOW STATUS LIKE 'mle%';</copy>
  ```

  **OUTPUT:** 
  ```
  +------------------------------+---------------+
  | Variable_name                | Value         |
  +------------------------------+---------------+
  | mle_heap_status              | Not Allocated |
  | mle_languages_supported      | JavaScript    |
  | mle_memory_used              | 0             |
  | mle_oom_errors               | 0             |
  | mle_session_resets           | 0             |
  | mle_sessions                 | 0             |
  | mle_sessions_max             | 0             |
  | mle_status                   | Inactive      |
  | mle_stored_functions         | 0             |
  | mle_stored_procedures        | 0             |
  | mle_stored_program_bytes_max | 0             |
  | mle_stored_program_sql_max   | 0             |
  | mle_stored_programs          | 0             |
  | mle_threads                  | 0             |
  | mle_threads_max              | 1             |
  +------------------------------+---------------+
  ```

8. Variables for MLE have the prefix 'mle\_'. We can control also the maximum amount of memory to allocate to the MLE component ('mle.memory\_max').  
  This variable is dynamic, but can be set only when the component is inactive (so at install time or before any use of JavaScript stored programs).  
  Memory is not allocated until the component is activated by creating or executing a stored program that uses JavaScript.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT format_bytes(@@mle.memory_max);</copy>
  ```

  **OUTPUT:** 
  ```
  +--------------------------------+
  | format_bytes(@@mle.memory_max) |
  +--------------------------------+
  | 1.27 GiB                       |
  +--------------------------------+
  ```

## Task 2: the first javascript store program

1. Now let's create and run our first stored function and check how the variables changes.  
  First set the test database as default

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>USE test;</copy>
  ```

2. The structure of javascript stored procedure and functions is similar to the ones created in SQL.  
  In this example we see  
  * CREATE FUNCTION <name>>(<arguments), where argumetns have the SQL data type
  * RETURN <datatype>, that indicate the return value
  * LANGUAGE JAVASCRIPT, that can be SQL (default) or JAVASCRIPT
  * AS <delimiter>, where <delimiter> marks the start and the end of the body
  * there is no ";" to separate JavaScript statements, because it's optional in javascript. So we don't need to specify a DELIMITER like for SQL store programming

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>
  CREATE FUNCTION add_int(arg1 INT, arg2 INT) 
  RETURNS INT DETERMINISTIC
  LANGUAGE JAVASCRIPT 
  AS $mle$
    return arg1 + arg2
  $mle$
  ;
  </copy>
  ```

  **OUTPUT:** 
  ```
  Query OK, 0 rows affected (0.0041 sec)
  ```

3. Execution is the same like for SQL store programs.
  ```sql
  <copy>SELECT add_int(3,7);</copy>
  ```

  **OUTPUT:** 
  ```
  +--------------+
  | add_int(3,7) |
  +--------------+
  |           10 |
  +--------------+
  ```

4. Please note that even if you employ the optional '**;**' character to separate JavaScript statements, this is interpreted correctly as being part of the JavaScript routine, and not as an SQL statement delimiter.  
  In this example we end the RETURN line with '**;**', without worries for the DELIMITER.

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>
  CREATE FUNCTION add_int2(arg1 INT, arg2 INT) 
  RETURNS INT DETERMINISTIC
  LANGUAGE JAVASCRIPT AS $mle$
    return arg1 + arg2;
  $mle$
  ;
  </copy>
  ```

  **OUTPUT:** 
  ```
  Query OK, 0 rows affected (0.0041 sec)
  ```

5. Execution is the same like for SQL store programs.

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT add_int2(3,7);</copy>
  ```

  **OUTPUT:** 
  ```
  +---------------+
  | add_int2(3,7) |
  +---------------+
  |            10 |
  +---------------+
  ```

6. We can now see how the MLE component status is changed from INACTIVE to ACTIVE

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT * FROM performance_schema.error_log WHERE SUBSYSTEM='MLE';</copy>
  ```

  **OUTPUT:** 
  ```
  +----------------------------+-----------+------+------------+-----------+-------------------------------------------+
  | LOGGED                     | THREAD_ID | PRIO | ERROR_CODE | SUBSYSTEM | DATA                                      |
  +----------------------------+-----------+------+------------+-----------+-------------------------------------------+
  | 2025-03-05 09:52:02.074877 |         9 | Note | MY-015000  | MLE       | Component starting...                     |
  | 2025-03-05 09:52:02.080259 |         9 | Note | MY-015000  | MLE       | Component state: INACTIVE                 |
  | 2025-03-05 10:51:14.884583 |         9 | Note | MY-015000  | MLE       | Component state: ACTIVE                   |
  | 2025-03-05 10:51:14.884644 |         9 | Note | MY-015000  | MLE       | Max memory 1363148800 produces -Xmx=525MB |
  +----------------------------+-----------+------+------------+-----------+-------------------------------------------+
  ```

7. We can also see that  
  * the memory is 'Allocated' (mle\_heap\_status)
  * we are using 23% of the memory (mle\_memory\_used)
  * we have 2 js functions in memory cache (mle\_stored\_functions) where the overall js functions and procedures is also 2 (mle\_stored\_programs)
  * there are no "out of memory errors" (mle\_oom\_errors)
  * there is only one sessions using mle (mle\_sessions)

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SHOW STATUS LIKE 'mle%';</copy>
  ```

  **OUTPUT:** 
  ```
  +------------------------------+------------+
  | Variable_name                | Value      |
  +------------------------------+------------+
  | mle_heap_status              | Allocated  |
  | mle_languages_supported      | JavaScript |
  | mle_memory_used              | 23         |
  | mle_oom_errors               | 0          |
  | mle_session_resets           | 0          |
  | mle_sessions                 | 1          |
  | mle_sessions_max             | 1          |
  | mle_status                   | Active     |
  | mle_stored_functions         | 2          |
  | mle_stored_procedures        | 0          |
  | mle_stored_program_bytes_max | 27         |
  | mle_stored_program_sql_max   | 0          |
  | mle_stored_programs          | 2          |
  | mle_threads                  | 1          |
  | mle_threads_max              | 1          |
  +------------------------------+------------+
  ```

8. Variables for MLE have the prefix 'mle\_'. The most  important now is 'mle.memory\_max', that set the maximum amount of memory to allocate to the MLE component.  
  This variable is dynamic, but can be set only when the component is inactive (so at install time or before any use of JavaScript stored programs).  
  Memory is not allocated until the component is activated by creating or executing a stored program that uses JavaScript.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT format_bytes(@@mle.memory_max);</copy>
  ```

  **OUTPUT:** 
  ```
  +--------------------------------+
  | format_bytes(@@mle.memory_max) |
  +--------------------------------+
  | 1.27 GiB                       |
  +--------------------------------+
  ```


You may now **proceed to the next lab**


## Learn More

* [MySQL Linux Installation](https://dev.mysql.com/doc/en/binary-installation.html)
* [MySQL tutorial](https://dev.mysql.com/doc/refman/8.4/en/tutorial.html)

## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025

