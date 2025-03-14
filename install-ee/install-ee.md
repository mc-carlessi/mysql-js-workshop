# INSTALL - MYSQL ENTERPRISE EDITION

## Introduction

Installation of MySQL Enterprise Edition 8 and MySQL Shell on Linux using RPMs.

MySQL Shell is an advanced client and code editor for MySQL. In addition to the provided SQL functionality, similar to mysql, MySQL Shell provides scripting capabilities for JavaScript and Python and includes APIs for working with MySQL.
In this lab we use the client as an alternative to the legacy mysql client.

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
  
## Task 1: Install MySQL Enterprise Edition using Linux RPM's


1. Connect to your **server** instance using an ssh client (**Example:** ssh -i ~/.ssh/id_rsa opc@132.145.17….)

  **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i ~/.ssh/id_rsa opc@<your_server_public_ip></copy>
    ```

    ![CONNECT](./images/ssh-login-2.png " ")


2. We have the required software available in **/workshop** directory. First we install the server

  **![green-dot](./images/green-square.jpg) shell>**  
      ```
      <copy>cd /workshop/linux/MySQL_server_innovation_rpms</copy>
      ```
  **![green-dot](./images/green-square.jpg) shell>**  
      ```
      <copy>ls -l</copy>
      ```

 **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>sudo yum -y install *.rpm</copy>
    ```

3. Now we install the MySQL Shell client You can see that the only packages are server and clients.

  **![green-dot](./images/green-square.jpg) shell>**  
      ```
      <copy>cd /workshop/linux/</copy>
      ```
  **![green-dot](./images/green-square.jpg) shell>**  
      ```
      <copy>ls -l</copy>
      ```

 **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>sudo yum -y install mysql-shell-commercial-9*.rpm</copy>
    ```

5.	Start the mysql instance

 **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>sudo systemctl start mysqld</copy>
    ```

6.	Verify that process is running and listening on the default ports (3306 for MySQL standard protocol and 33060 for MySQL XDev protocol)

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>sudo systemctl status mysqld</copy>
    ```

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>netstat -an | grep 3306</copy>
    ```

7.	Another way is searching the message “ready for connections” in error log as one of the last 

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>sudo grep -i ready /var/log/mysqld.log</copy>
    ```

8.	Retrieve root password for first login:

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>sudo grep -i 'temporary password' /var/log/mysqld.log</copy>
    ```

9. Login to the the mysql-enterprise, change temporary password and check instance the status

    **![green-dot](./images/green-square.jpg) shell>** 
     ```
    <copy>mysqlsh root@localhost</copy>
    ```

10. Create New Password for MySQL Root

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>ALTER USER 'root'@'localhost' IDENTIFIED BY 'Welcome1!';</copy>
    ```

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\status</copy>
    ```

## Task 2: Create a new administrative user and import a sample database

1.	Create a new administrative user called 'admin' with remote access and full privileges

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>CREATE USER 'admin'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;</copy>
    ```

2. Test the login of the new user (saving the password).

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
  <copy>mysqlsh admin@127.0.0.1</copy>
  ```

3. Import the world sample database that is in /workshop/databases folder.

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>SOURCE /workshop/databases/world/world.sql;</copy>
    ```

  > **IMPORTANT NOTE**: MySQL shell is a client for SQL language or command mode to manage various activities in your MySQL instance.  
  The two command mode available are in javascript and python. Don't be confused: javascript command mode is used to manage replicaSets, InnODB clusters, dump& load, etc. so it's not part of this workshop.

4. To exit from MySQL Shell, use '\quit' or '\q'
  
  **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\quit</copy>
    ```

You can now **proceed to the next lab**.

## Learn More

* [MySQL Linux Installation](https://dev.mysql.com/doc/en/binary-installation.html)
* [MySQL tutorial](https://dev.mysql.com/doc/refman/8.4/en/tutorial.html)

## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025

