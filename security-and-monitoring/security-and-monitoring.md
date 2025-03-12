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

## Task 2: Store programs information

1. We see now some useful queries.  
  But first, reconnect as admin

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>\c admin@127.0.0.1</copy>
  ```

2. We can search all the programs in a specific database

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>select ROUTINE_NAME from information_schema.routines where ROUTINE_SCHEMA='test';</copy>
  ```

  **OUTPUT SAMPLE:**
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

3. We can show the creation code of the program.  
  Be careful to choose correctly between 'SHOW CREATE **PROCEDURE**' and 'SHOW CREATE **FUNCTION**'  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SHOW CREATE FUNCTION helloword_jsf\G</copy>
  ```

  **OUTPUT:**
  ```
  *************************** 1. row ***************************
             Function: helloword_jsf
             sql_mode: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
      Create Function: CREATE DEFINER=`admin`@`%` FUNCTION `helloword_jsf`(name VARCHAR(50)) RETURNS char(50) CHARSET utf8mb4
      DETERMINISTIC
      LANGUAGE JAVASCRIPT
  AS $$
      return "Hello world from " + name;
    $$
  character_set_client: utf8mb4
  collation_connection: utf8mb4_0900_ai_ci
    Database Collation: utf8mb4_0900_ai_ci
  ```

4. We can check the characteristics of a stored procedure, such as the database, name, type, creator, creation and modification dates, and character set information with "SHOW [PROCEDURE|FUNCTION] STATUS &lt;LIKE 'program name'&gt;"  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SHOW FUNCTION STATUS LIKE 'helloword_jsf'\G</copy>
  ```

  **OUTPUT:**
  ```
  *************************** 1. row ***************************
                    Db: test
                  Name: helloword_jsf
                  Type: FUNCTION
              Language: JAVASCRIPT
               Definer: admin@%
              Modified: 2025-03-05 17:19:37
               Created: 2025-03-05 17:19:37
         Security_type: DEFINER
               Comment:
  character_set_client: utf8mb4
  collation_connection: utf8mb4_0900_ai_ci
    Database Collation: utf8mb4_0900_ai_ci
  ```

5. We can check just the parameters  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>
  SELECT * 
  FROM information_schema.parameters 
  WHERE SPECIFIC_SCHEMA='test' AND SPECIFIC_NAME='helloword_jsf'\G
  </copy>
  ```

  **OUTPUT:**
  ```
  *************************** 1. row ***************************
          SPECIFIC_CATALOG: def
           SPECIFIC_SCHEMA: test
             SPECIFIC_NAME: helloword_jsf
          ORDINAL_POSITION: 0
            PARAMETER_MODE: NULL
            PARAMETER_NAME: NULL
                 DATA_TYPE: char
  CHARACTER_MAXIMUM_LENGTH: 50
    CHARACTER_OCTET_LENGTH: 200
         NUMERIC_PRECISION: NULL
             NUMERIC_SCALE: NULL
        DATETIME_PRECISION: NULL
        CHARACTER_SET_NAME: utf8mb4
            COLLATION_NAME: utf8mb4_0900_ai_ci
            DTD_IDENTIFIER: char(50)
              ROUTINE_TYPE: FUNCTION
  *************************** 2. row ***************************
          SPECIFIC_CATALOG: def
           SPECIFIC_SCHEMA: test
             SPECIFIC_NAME: helloword_jsf
          ORDINAL_POSITION: 1
            PARAMETER_MODE: IN
            PARAMETER_NAME: name
                 DATA_TYPE: varchar
  CHARACTER_MAXIMUM_LENGTH: 50
    CHARACTER_OCTET_LENGTH: 200
         NUMERIC_PRECISION: NULL
             NUMERIC_SCALE: NULL
        DATETIME_PRECISION: NULL
        CHARACTER_SET_NAME: utf8mb4
            COLLATION_NAME: utf8mb4_0900_ai_ci
            DTD_IDENTIFIER: varchar(50)
              ROUTINE_TYPE: FUNCTION
  ```

4. We can also check the overall information related to a specific program interrogating the table information_schema.routines  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT * 
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

## Task 3: Libraries information

1. We can search all the libraries available  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT LIBRARY_SCHEMA, LIBRARY_NAME, LANGUAGE, CREATOR FROM information_schema.libraries;</copy>
  ```

  **OUTPUT SAMPLE:**
  ```
  +----------------+-----------------+------------+---------+
  | LIBRARY_SCHEMA | LIBRARY_NAME    | LANGUAGE   | CREATOR |
  +----------------+-----------------+------------+---------+
  | jslib          | lib1            | JAVASCRIPT | admin@% |
  | test           | my_math_library | JAVASCRIPT | admin@% |
  +----------------+-----------------+------------+---------+
  ```

3. We can show the creation code of the library.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SHOW CREATE LIBRARY jslib.lib1\G</copy>
  ```

  **OUTPUT:**
  ```
  *************************** 1. row ***************************
         Library: lib1
        sql_mode: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
  Create Library: CREATE LIBRARY `lib1`
      LANGUAGE JAVASCRIPT
  AS $$
      export function f(n) {
        return n * 2;
      }
    $$
  ```

This **ends the workshop**


## Learn More

* [MySQL Linux Installation](https://dev.mysql.com/doc/en/binary-installation.html)
* [MySQL tutorial](https://dev.mysql.com/doc/refman/8.4/en/tutorial.html)

## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025

