# MYSQL JAVASCRIPT LIBRARIES  

## Introduction
One opf the benefits of Javascript is the option to create and reuse libraries.  
JavaScript language support in MySQL conforms to the [ECMAscript 2023](https://262.ecma-international.org/14.0/).  

> Note 
  * Neither file access nor network access from JavaScript stored program code is supported.
  * There is no support for Node.js
  * Library functions can be invoked only within the library or stored routine into which their containing library is imported.
  * MLE JavaScript library code is executed only when invoked as part of a stored routine (not by CREATE FUNCTION,CREATE PROCEDURE or CREATE LIBRARY)


Goal:
- Create Javascript libraries
- Access Javascript functions inside libraries

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

## Task 1: Create Javascript libraries

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```shell
    <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
    ```

2. Now connect yo your MySQL instance

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```sql
    <copy>mysqlsh root@localhost</copy>
    ```

3. Let's create a database to store our libraries

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```sql
    <copy>CREATE DATABASE IF NOT EXISTS jslib;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```sql
    <copy>USE jslib;</copy>
    ```

4. A javascript library can be created with 'CREATE LIBRARY' statement.  
  We create here
  * a library named lib1
  * a simple function that return double the number that you specify

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```js
  <copy>
  CREATE LIBRARY IF NOT EXISTS jslib.lib1 
  LANGUAGE JAVASCRIPT AS $mle$
    export function f(n) {
      return n * 2;
    }
  $mle$;
  </copy>
  ```

## Task 2: Use Javascript functions inside libraries

1. Library functions can be invoked only within the library or stored routine into which their containing library is imported.
  To import a library into your function, use 'USING'.  The keyword 'AS ...? is option, but can be used to simplify the code reading

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```js
  <copy>
  CREATE FUNCTION myfunc1(n INT) RETURNS INT DETERMINISTIC
  LANGUAGE JAVASCRIPT
  USING (jslib.lib1 AS mylib)
  AS $mle$
    return mylib.f(n);
  $mle$;
  </copy>
  ```

2. Now execute the function that uses the library

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT myfunc1(3), myfunc1(4), myfunc1(5);</copy>
  ```

  **OUTPUT:** 
  ```text
  +------------+------------+------------+
  | myfunc1(3) | myfunc1(4) | myfunc1(5) |
  +------------+------------+------------+
  |          6 |          8 |         10 |
  +------------+------------+------------+
  ```

3. JavaScript syntax is checked at library creation time.  
  Here we used '$' insted of '*'

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>
  CREATE LIBRARY IF NOT EXISTS jslib.lib2 LANGUAGE JAVASCRIPT
    AS $mle$
      export function f(n) {
        return n $ 2
      }
    $mle$;
  </copy>
  ```

  **OUTPUT:** 
  ```text
  ERROR: 6113 (HY000): JavaScript> SyntaxError: lib2:3:17 Expected ; but found $
          return n $ 2
                  ^
  ```

You can now **proceed to the next lab**


## Learn More

* [MySQL Shell manual](https://dev.mysql.com/doc/mysql-shell/8.4/en/)
* [mysql\_config\_editor manual page](https://dev.mysql.com/doc/refman/8.4/en/mysql-config-editor.html)

## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025
