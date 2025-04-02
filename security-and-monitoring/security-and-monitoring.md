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

## Task 1: Stored programs executed with invoker privilege

1. Connect to your **server** instance using your web browser

  ![CONNECT](./images/login-with-labels.png)  


2. Now connect yo your MySQL instance

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>mysqlsh root@localhost</copy>
  ```

3. By default, stored programs are executed with DEFINER (who create it) privileges.  
  This may be a risk for the security, so it's a best practice to define the store programs to use the INVOKER (who call/execute the stored program) privilege using 'SQL SECURITY' specification.
  Let's recreate the stored procedure that return the cities with more than 1 million people.

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```js
  <copy>
  CREATE PROCEDURE test.cities_1million_secure(arg1 CHAR(3))
  SQL SECURITY INVOKER
  LANGUAGE JAVASCRIPT AS $mle$
    // arg1 is an existing country code inside world.city database
    let myquery='SELECT Name, Population FROM world.city WHERE Population > 1000000 AND CountryCode = "' + arg1 + '"';
    console.clear();

    let s = session.sql(myquery);
    
    let res = s.execute();
    
    console.log(res.getColumnNames());
    
    let row = res.fetchOne();
    
    while(row) {
      console.log(row.toArray());
      row = res.fetchOne();
    }
  $mle$;
  ```

4. Let's now create a new user.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>CREATE USER appuser@'%' IDENTIFIED BY 'Welcome1!';</copy>
  ```

5. And give him only the privilege to execute 'cities\_1million\secure'.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>GRANT EXECUTE ON PROCEDURE test.cities_1million_secure to appuser@'%';</copy>
  ```

6. Now switch to the new user

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

7. If we execute hte stored procedure, are used hte privilege of the user, so it return an error  
  We don't risk to share information that are not visible to the user.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>CALL test.cities_1million_secure('ITA');</copy>
  ```

  **OUTPUT:**
  ```
  ERROR: 1142 (42000): SELECT command denied to user 'appuser'@'localhost' for table 'city'
  ```


## Task 2: Stored programs to limit data access

1. Of course, we can use the DEFINER/INVOKER to grant controlled access to data.  
  Let's reconnect as admin user

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>\c admin@127.0.0.1</copy>
  ```

2. As you remember, we already created a function that returns the cities with more than one million people.
  Let's compare the execution context for 'cities\_1million' and 'cities\_1million_secure'.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>
  SELECT ROUTINE_SCHEMA, ROUTINE_NAME, DEFINER, SECURITY_TYPE 
  FROM information_schema.routines 
  WHERE ROUTINE_SCHEMA='test' AND ROUTINE_NAME like 'cities_1million%';
  </copy>
  ```

  **OUTPUT:**
  ```
  +----------------+------------------------+---------+---------------+
  | ROUTINE_SCHEMA | ROUTINE_NAME           | DEFINER | SECURITY_TYPE |
  +----------------+------------------------+---------+---------------+
  | test           | cities_1million        | admin@% | DEFINER       |
  | test           | cities_1million_secure | admin@% | INVOKER       |
  +----------------+------------------------+---------+---------------+
  ```

3. We now want that appuser is able to retrieve the cities with more than 1 million people, without grant him privileges on world database.  
  So we grant the execution privilege of the 'cities\_1million' procedure to him.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>GRANT EXECUTE ON PROCEDURE test.cities_1million to appuser@'%';</copy>
  ```

4. Now switch to the new user

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>\c appuser@127.0.0.1</copy>
  ```

6. User is still unable to read data from world.city tables.    

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT * from world.city LIMIT 5;</copy>
  ```

  **OUTPUT:**
  ```
  ERROR: 1142 (42000): SELECT command denied to user 'appuser'@'localhost' for table 'city'
  ```

7. But the user is able to retrieve the required information using the stored procedure

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>CALL test.cities_1million('ITA');</copy>
  ```

  **OUTPUT:**
  ```
  Query OK, 0 rows affected (0.0029 sec)
  ```

8. And of course see the output   

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

## Task 3: Stored programs information

1. We see now some useful queries.  
  But first, reconnect as admin

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>\c admin@127.0.0.1</copy>
  ```

2. We can search all the programs in a specific database

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT ROUTINE_NAME FROM information_schema.routines WHERE ROUTINE_SCHEMA='test';</copy>
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

## Task 4: Libraries information

1. We can search all the libraries available  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT LIBRARY_SCHEMA, LIBRARY_NAME, LANGUAGE, CREATOR FROM information_schema.libraries;</copy>
  ```

  **OUTPUT SAMPLE:**
  ```
  +----------------+------------------+------------+---------+
  | LIBRARY_SCHEMA | LIBRARY_NAME     | LANGUAGE   | CREATOR |
  +----------------+------------------+------------+---------+
  | jslibs         | lib1             | JAVASCRIPT | admin@% |
  | test           | ext_math_library | JAVASCRIPT | admin@% |
  +----------------+------------------+------------+---------+
  ```

3. We can show the creation code of the library.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SHOW CREATE LIBRARY jslibs.lib1\G</copy>
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

This **ends the workshop**.


## Learn More

* [Stored Object Access Control](https://dev.mysql.com/doc/refman/9.2/en/stored-objects-security.html)


## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025

