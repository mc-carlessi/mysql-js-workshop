# MYSQL JAVASCRIPT STORED PROGRAMMING  

## Introduction
MySQL store programming is traditionally based on stored procedures and functions.  
We see her how to create them.

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

## Task 1: Javascript Store procedures

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```shell
    <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
    ```

2. Now connect yo your MySQL instance

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```sql
    </span> <copy>mysqlsh admin@127.0.0.1</copy>
    ```

3. Let's set the default database for our store programs

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```sql
    <copy>USE test;</copy>
    ```

4. A javascript **stored procedure** (like SQL stored procedures) is an object created with CREATE PROCEDURE and invoked using the CALL statement. This object contains a set of javascript instructions that are executed as a program.  
  A procedure **does not have a return value**, but can modify its parameters for later inspection by the caller. It can also generate result sets to be returned to the client program.  
  Let's recap
  * CREATE PROCEDURE <name of the procedure>()
  * LANGUAGE JAVASCRIPT, that can be SQL (default) or JAVASCRIPT
  * AS <delimiter>, where 'delimiter' marks the start and the end of the body (instead of BEGIN...END)
  * The usual console.log() function that is commonly used in javascript to print results in the output 

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```js
  <copy>
  CREATE PROCEDURE helloword_js() 
  LANGUAGE JAVASCRIPT 
  AS $mle$
    console.log("Hello World");
  $mle$
  ;
  </copy>
  ```

5. Our first stored procedure is created, and we are now ready to execute it using the 'CALL' command.
  Unfortunately the result is not what expected: no visible output.

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**

  ```sql
  <copy>CALL helloword_js;</copy>
  ```

  **OUTPUT:** (time is different in your execution)
  ```
  Query OK, 0 rows affected (0.0072 sec)
  ```

6. The reason for above behavior is that javascript stadin, stout, sterr are manged by mle.
  To see our output we need to ask mle to show it. The function is mle\_session\_state() that accept different parameters (we see here just a couple).

  * To show the output

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT mle_session_state("stdout") AS 'STDOUT';</copy>
  ```

  **OUTPUT:**
  ```
  +--------------+
  | STDOUT       |
  +--------------+
  | Hello World
  |
  +--------------+
  ```

  * To show the standard error

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT mle_session_state("stderr") AS 'STDERR';</copy>
  ```

  **OUTPUT:**
  ```
  +--------+
  | STDERR |
  +--------+
  |        |
  +--------+
  ```

## Task 2: Javascript stored functions

1. A javascript **stored function** is an object created with CREATE FUNCTION and invoked like a native function. This object contains a set of javascript instructions that are executed as a program.  
  A function can **return value** and can have parameters.   
  Now we create a simple one specifying
  * CREATE FUNCTION <name of the function>()
  * LANGUAGE JAVASCRIPT, that can be SQL (default) or JAVASCRIPT
  * AS <delimiter>, where 'delimiter' marks the start and the end of the body (instead of BEGIN...END)
  * Return an "Hello World" greeting 

**![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```js
  <copy>
  CREATE FUNCTION helloword_jsf(name VARCHAR(50)) RETURNS CHAR(50) DETERMINISTIC
  LANGUAGE JAVASCRIPT
  AS $mle$
    return "Hello world from " + name;
  $mle$
  ;
  </copy>
  ```

2. Check the result with an empty string

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT helloword_jsf('');</copy>
  ```

  **OUTPUT:**
  ```
  +-------------------+
  | helloword_jsf('') |
  +-------------------+
  | Hello world from  |
  +-------------------+
  ```

3. Check the result with a value for the parameter

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT helloword('Goofy') AS message;</copy>
  ```

  **OUTPUT:**
  ```
  +-----------------------------+
  | message                     |
  +-----------------------------+
  | Hello world from from Goofy |
  +-----------------------------+
  ```

4. To control the flow you can use the stardard if...then...else, while, etc. statements.  
  We can now rewrite he the greatest common divisor function from SQL to javascript.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```js
  <copy>
  DROP FUNCTION IF EXISTS gcd_js;

  CREATE FUNCTION gcd_js(x int, y int) RETURNS int DETERMINISTIC
  LANGUAGE JAVASCRIPT AS $mle$
    let dividend=1;
    let divisor=1;
    let remainder=1;

    // Return the larger and the smaller, so the MOD operation make sense
    dividend = (x > y) ? x : y ;
    remainder = (x < y) ? x : y ;

    while (remainder !== 0) {
      divisor = remainder;
      remainder = dividend % divisor;
      dividend = divisor;
    }
    
    return divisor;
  $mle$
  ;
  </copy>
  ```

6. Check the result with a value for the parameter

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT gcd_js(10,5);</copy>
  ```

  **OUTPUT:**
  ```
  +--------------+
  | gcd_js(10,5) |
  +--------------+
  |            5 |
  +--------------+
  ```

7. Of course we write the procedure in a way ore in line with javascript.  
  We use here a recursive function, and the mod functions that doesn't care which number is greatest.
  Please remember that by default the maximum recursion is 1000. 

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```js
  <copy>
  CREATE FUNCTION gcd_js_short(arg1 INT, arg2 INT) RETURNS INT DETERMINISTIC
  LANGUAGE JAVASCRIPT AS $mle$
    let gcd_js_rec = (a, b) => b ? gcd_js_rec(b, a % b) : Math.abs(a)
    return gcd_js_rec(arg1, arg2)
  $mle$
  ;
  ```

6. Check the result with a value for the parameter

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```sql
  <copy>SELECT gcd_js_short(10,5);</copy>
  ```

  **OUTPUT:**
  ```
  +--------------------+
  | gcd_js_short(10,5) |
  +--------------------+
  |                  5 |
  +--------------------+
  ```

## Task 3: Javascript and SQL API

> Note: The SQL API can be used only within JavaScript stored procedures, and cannot be used within JavaScript stored functions  

1. Let's write a store procedure that return cities of a country wit more that 1M inhabitants.  
  It looks too simple at first sight, but we it help us understand better the usage and it offers a good opportunity to explain security later on.
  Here the steps to use an SQL query inside javascript
  * use the variable "myquery" to build the query
  * we insert a comment using <code>//</code> to clarify the arg1 usage
  * clear the console stdout (like mle\_session\_reset() but only for stdout and stderr)
  * creates the session object for the query with session.sql()
  * execute the query with &lt;session_object&gt;.execute and put the result in 'res' variable
  * write the column name of the row ()
  * now fetch all the rows one by one and print them in stdout (the toArray() is a javascript function that transforms array-like objects into real arrays)  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
  ```js
  <copy>
  CREATE PROCEDURE cities_1million(arg1 CHAR(3))
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

2. Now we invoke the store procedure.

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**

  ```sql
  <copy>CALL cities_1million('ITA');</copy>
  ```

  **OUTPUT:** (time is different in your execution)
  ```
  Query OK, 0 rows affected (0.0072 sec)
  ```

3. We can now check the output

  * To show the output

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

## Task 4: Javascript error handling

> NOTE: Some errors cannot be handled in JavaScript. For example, if a query is aborted (CTRL-C), the stored program stops executing immediately and produces an error. Likewise, out of memory errors cannot be handled within JavaScript routines.

1. An SQL statement that causes errors that are not handled within the stored program passes them back to the client.  
  To observe this, we create a stored procedure using the following SQL statement:

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```js
  <copy>
  CREATE PROCEDURE jssp_simple_error(IN mytable VARCHAR(250))
  LANGUAGE JAVASCRIPT AS $mle$
    let session = mysql.getSession()
    let res = session.sql("SELECT * FROM " + mytable + " LIMIT 5;").execute()
    console.clear();

    console.log(res.getColumnNames());
    let row = res.fetchOne();
    
    while(row) {
      console.log(row.toArray());
      row = res.fetchOne();
    }
  $mle$;
  </copy>
  ```

2. Now we call jssp\_simple\_error(), passing to it a table in the world database

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>CALL jssp_simple_error("world.city");</copy>
  ```

  **OUTPUT:**
  ```
  Query OK, 0 rows affected (0.0028 sec)
  ```

3. And check the result.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT mle_session_state("stdout") AS 'STDOUT';</copy>
  ```

  **OUTPUT:** (it can be a bit different)
  ```text
  +----------------------------------------------------------+
  | mle_session_state("stdout")                              |
  +----------------------------------------------------------+
  | ID,Name,CountryCode,District,Population
  1,Kabul,AFG,Kabol,1780000
  2,Qandahar,AFG,Qandahar,237500
  3,Herat,AFG,Herat,186800
  4,Mazar-e-Sharif,AFG,Balkh,127800
  5,Amsterdam,NLD,Noord-Holland,731200
   |
  +----------------------------------------------------------+
  ```

4. But if we ask for a non existent table the procedure stop suddenly with an error.  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>CALL jssp_simple_error("t_unknown");</copy>
  ```

  **OUTPUT:**
  ```
  ERROR: 1146 (42S02): Table 'test.t_unknown' doesn't exist
  ```

5. It's better top handle SQL errors using try-catch syntax.
  * put the code execution inside the try block
  * catch errors inside the variable e
  * execute a meaningful execution is case of error, for example with a message 

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```js
  <copy>
  CREATE PROCEDURE jssp_simple_error(IN mytable VARCHAR(250))
  LANGUAGE JAVASCRIPT AS $mle$
  let session = mysql.getSession()
  console.clear();

  try {
    let res = session.sql("SELECT * FROM " + mytable + " LIMIT 5;").execute()

    console.log(res.getColumnNames());
    let row = res.fetchOne();
    
    while(row) {
      console.log(row.toArray());
      row = res.fetchOne();
      }
    } catch (e) {
      console.error("\nJS Error:\n" + e.name + ":\n" + e.message)
    }

  $mle$;
  </copy>
  ```

6. Let's retry with an existing table.  
  Because the table exists, the instructions inside try blocka re executed  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>CALL jssp_simple_error("world.city");</copy>
  ```

  **OUTPUT:**
  ```
  Query OK, 0 rows affected (0.0028 sec)
  ```

7. We check here the stdout and stderr, with a different line terminator to simplify the reading.    

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT mle_session_state("stdout") AS 'STDOUT', mle_session_state("stderr") AS 'STDERR'\G</copy>
  ```

  **OUTPUT:** 
  ```text
  *************************** 1. row ***************************
  STDOUT: ID,Name,CountryCode,District,Population
  1,Kabul,AFG,Kabol,1780000
  2,Qandahar,AFG,Qandahar,237500
  3,Herat,AFG,Herat,186800
  4,Mazar-e-Sharif,AFG,Balkh,127800
  5,Amsterdam,NLD,Noord-Holland,731200

  STDERR:
  1 row in set (0.0002 sec)
  ```

8. Now, if we ask for a non existent table the procedure continue to works...  

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>CALL jssp_simple_error("t_unknown");</copy>
  ```

  **OUTPUT:**
  ```
  Query OK, 0 rows affected (0.0009 sec)
  ```

9. ...but an error message is sent to stderr.    

  **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
  ```sql
  <copy>SELECT mle_session_state("stdout") AS 'STDOUT', mle_session_state("stderr") AS 'STDERR'\G</copy>
  ```

  **OUTPUT:** 
  ```text
  *************************** 1. row ***************************
  STDOUT:
  STDERR:
  JS Error:
  org.graalvm.polyglot.nativeapi.PolyglotNativeAPI$CallbackException:
  SQL-CALLOUT: Error code: 1146 Error state: 42S02 Error message: Table 'test.t_unknown' doesn't exist

  1 row in set (0.0002 sec)
  ```


## Learn More

* [MySQL Shell manual](https://dev.mysql.com/doc/mysql-shell/8.4/en/)
* [mysql\_config\_editor manual page](https://dev.mysql.com/doc/refman/8.4/en/mysql-config-editor.html)

## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025
