# MYSQL JAVASCRIPT - SECURITY AND MONITORING

## Introduction
Stored programs (procedures, functions, triggers, and events) and views are defined prior to use and, when referenced, execute within a security context that determines their privileges. The privileges applicable to execution of a stored object are controlled by its DEFINER attribute and SQL SECURITY characteristic.  
* Creating a stored object with a nonexistent DEFINER account creates an orphan object, that may introduce errors or unexpected behavior.  
  The default object definer is the user who creates it.  
* The object definition can include an SQL SECURITY characteristic with a value of DEFINER or INVOKER to specify whether the object executes in definer or invoker context.  
  The default is definer context.

In addition, you may want to list the stored procedures, or see the definitions, and other info.

Estimated Time: 15 minutes

### Objectives

In this lab, you will:

* Play with the security model
* Execute monitoring queries


### Prerequisites

This lab assumes you have:
* Completed previous labs

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

3. A stored object that executes in definer security context executes with the privileges of the account named by its DEFINER attribute.  
  When you create a stored program, the default definer is who create the program.  
  We created the store programs as admin, so let's now create a new user.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>CREATE USER appuser@'%' IDENTIFIED BY 'Welcome1!';</copy>
  ```

4. Now we can grant the execution privilege of the 'cities\_1million' procedure to the new user.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>GRANT EXECUTE ON PROCEDURE test.cities_1million to appuser@'%';</copy>
  ```

5. Now switch to the new user

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>\c appuser@127.0.0.1</copy>
  ```

6. And try to select data from world.city tables. Of course it doesn't works  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT * from world.city LIMIT 5;</copy>
  ```

  **OUTPUT:**
  ```
  ERROR: 1142 (42000): SELECT command denied to user 'appuser'@'localhost' for table 'city'
  ```

7. But the stored procedure was created with default DEFINER privilege

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>CALL test.cities_1million('ITA');</copy>
  ```

  **OUTPUT:**
  ```
  Query OK, 0 rows affected (0.0029 sec)
  ```

8. And appuser is able to see the result.  
  We limited the access to only the procedure and not to the full database!

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT mle_session_state("stdout") AS 'STDOUT';</copy>
  ```

  **OUTPUT:**
  ```
  +-------------------------------------------------------------+
  | STDOUT                                                      |
  +-------------------------------------------------------------+
  | Name,Population
  Roma,2643581
  Milano,1300977
  Napoli,1002619
  |
  +-------------------------------------------------------------+
  ```

## Task 2: Store procedure information

1. We see now some useful queries.  
  But first, reconnect as admin

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>\c admin@127.0.0.1</copy>
  ```

2. We can search the programs in a specific database

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>select ROUTINE_NAME from information_schema.routines where ROUTINE_SCHEMA='test';</copy>
  ```

  **OUTPUT:**
  ```
  +--------------------+
  | ROUTINE_NAME       |
  +--------------------+
  | add_int            |
  | add_int2           |
  | gcd_js             |
  | gcd_js2            |
  | gcd_js_short       |
  ...
  ```

3. We can also check the content of a specific program

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>select ROUTINE_NAME ROUTINE_TYPE, CREATED, LAST_ALTERED, ROUTINE_COMMENT, ROUTINE_DEFINITION 
  FROM information_schema.routines 
  WHERE ROUTINE_SCHEMA='test' 
  AND ROUTINE_NAME='helloword_jsf'\G</copy>
  ```

  **OUTPUT:**
  ```
  *************************** 1. row ***************************
            SPECIFIC_NAME: helloword_jsf
          ROUTINE_CATALOG: def
            ROUTINE_SCHEMA: test
              ROUTINE_NAME: helloword_jsf
              ROUTINE_TYPE: FUNCTION
                DATA_TYPE: char
  CHARACTER_MAXIMUM_LENGTH: 50
    CHARACTER_OCTET_LENGTH: 200
        NUMERIC_PRECISION: NULL
            NUMERIC_SCALE: NULL
        DATETIME_PRECISION: NULL
        CHARACTER_SET_NAME: utf8mb4
            COLLATION_NAME: utf8mb4_0900_ai_ci
            DTD_IDENTIFIER: char(50)
              ROUTINE_BODY: EXTERNAL
        ROUTINE_DEFINITION:
      return "Hello world from " + name;

            EXTERNAL_NAME: NULL
        EXTERNAL_LANGUAGE: JAVASCRIPT
          PARAMETER_STYLE: SQL
          IS_DETERMINISTIC: YES
          SQL_DATA_ACCESS: CONTAINS SQL
                  SQL_PATH: NULL
            SECURITY_TYPE: DEFINER
                  CREATED: 2025-03-05 17:19:37
              LAST_ALTERED: 2025-03-05 17:19:37
                  SQL_MODE: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
          ROUTINE_COMMENT:
                  DEFINER: admin@%
      CHARACTER_SET_CLIENT: utf8mb4
      COLLATION_CONNECTION: utf8mb4_0900_ai_ci
        DATABASE_COLLATION: utf8mb4_0900_ai_ci
  ```

select ROUTINE_NAME ROUTINE_TYPE, CREATED, LAST_ALTERED, ROUTINE_COMMENT, ROUTINE_DEFINITION from information_schema.routines where ROUTINE_SCHEMA='test' and ROUTINE_NAME='helloword'\G






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

