# CLIENT-SERVER ARCHITECTURE WITH MYSQL

Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another.
In their communication, each machine has its own role: the machine sending requests is usually referred as "Client" and the machine responding (serving) is called "Server".
A simple diagram of Web Client-Server architecture is presented below:

![client-server-architecture](https://github.com/ifydevops23/Application_Architecture/assets/126971054/b8cc5f7f-51e6-4e6c-95a7-bafe7c47cc5c)
Assuming that you go on your browser, and typed in there [facebook.com.] It means that your browser is considered the "Client". Essentially, it is sending requests to the remote server, and in turn, would be expecting some kind of response from the remote server.
Let’s take a very quick example and see Client-Server communicatation in action.

Open up your Ubuntu or Windows terminal and run the curl command: <br>
`curl -Iv facebook.com`

In this example, your terminal will be the client, while [facebook.com] will be the server.
See the response from the remote server in the below output. You can also see that the requests from the URL are being served by a computer with an IP address on port 80.

![1_curl_from_facebook](https://github.com/ifydevops23/Application_Architecture/assets/126971054/b431e8b9-7eb2-4601-a276-9fb3c6060d9f)
Another simple way to get a server’s IP address is to use a simple diagnostic tool like ‘ping’. 

## IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS).

To demonstrate a basic client-server using MySQL Relational Database Management System (RDBMS),

Create and configure two Linux-based virtual servers (EC2 instances in AWS).
Server A name - `mysql server` <br>
Server B name - `mysql client`

![1_instances_created](https://github.com/ifydevops23/Application_Architecture/assets/126971054/245d706d-e6ec-4657-b5cc-0e555234a353)

- Install MySQL Server software on the VM called "mysql server".

![1_install_mysql_server](https://github.com/ifydevops23/Application_Architecture/assets/126971054/8e974d0c-9099-4c22-acbb-ed30be2ece91)
- Log in by typing `sudo mysql`
- Add Password to 'root' user of mysql server.

![1_alter_password_mysql](https://github.com/ifydevops23/Application_Architecture/assets/126971054/6c368084-5e23-4469-b69a-d75b4957cfd7)
- Logout by typing `exit`
- Log back in by typing `mysql -p`
- Create database 'my_database' CREATE DATABASE my_database;
- Create new user other than 'root' CREATE USER my_db_user;

![db_user_and_db](https://github.com/ifydevops23/Application_Architecture/assets/126971054/a5d4855a-3251-44ee-93b0-f2ce19904d19)
- Grant access to 'my_db_user' by typing `GRANT ALL ON my_database.* TO 'my_db_user'@'%';`
- Exit the mysql shell `mysql -u example_user -p`
- Login using new user to ascertain priviledges and permisiions <br>
  `mysql -u my_db_user -p`
- Query Database by typing `SHOW DATABASES;` 
- Exit shell `exit`


On mysql client Linux Server install MySQL Client software.
By default, both of your EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. For extra security, do not allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’.
You might need to configure MySQL server to allow connections from remote hosts.
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
Replace ‘127.0.0.1’ to ‘0.0.0.0’ like this:

From mysql client Linux Server connect remotely to mysql server Database Engine without using SSH. You must use the mysql utility to perform this action.
Check that you have successfully connected to a remote MySQL server and can perform SQL queries:
Show databases;
If you see an output similar to the below image, then you have successfully completed this project – you have deployed a fully functional MySQL Client-Server set up.
Well Done! You are getting there gradually. You can further play around with this set up and practice in creating/dropping databases & tables and inserting/selecting records to and from them.
Congratulations!

