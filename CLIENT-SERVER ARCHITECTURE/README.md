# CLIENT-SERVER ARCHITECTURE WITH MYSQL

Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another.
In their communication, each machine has its own role: the machine sending requests is usually referred as "Client" and the machine responding (serving) is called "Server".
A simple diagram of Web Client-Server architecture is presented below:

![client-server-architecture](https://github.com/ifydevops23/Application_Architecture/assets/126971054/b8cc5f7f-51e6-4e6c-95a7-bafe7c47cc5c) <br>
Assuming that you go on your browser, and typed in there [facebook.com.] It means that your browser is considered the "Client". Essentially, it is sending requests to the remote server, and in turn, would be expecting some kind of response from the remote server.
Let’s take a very quick example and see Client-Server communicatation in action.

Open up your Ubuntu or Windows terminal and run the curl command: <br>
`curl -Iv facebook.com`

In this example, your terminal will be the client, while [facebook.com] will be the server.
See the response from the remote server in the below output. You can also see that the requests from the URL are being served by a computer with an IP address on port 80.

![1_curl_from_facebook](https://github.com/ifydevops23/Application_Architecture/assets/126971054/b431e8b9-7eb2-4601-a276-9fb3c6060d9f)
Another simple way to get a server’s IP address is to use a simple diagnostic tool like ‘ping’. 

# STEP 0 - PREREQUISITES
- Two (2) VMs with Ubuntu OS named "mysql server" and "mysql client" 
- SSH Client
- Knowledge of MYSQL 

## STEP 1 - IMPLEMENT A CLIENT-SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS).

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
- Exit the mysql shell `exit`
- Login using new user to ascertain priviledges and permisiions <br>
  `mysql -u my_db_user -p`
- Query Database by typing `SHOW DATABASES;` 
- Exit shell `exit`

### Allowing Access from Client.
- Open TCP port 3306 (MySQL server port by default), by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups.<br>
  For extra security, do not allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’.

  ![2_update_security_group_(mysql_server_port)](https://github.com/ifydevops23/Application_Architecture/assets/126971054/16cfe24b-d468-451c-89bb-f509aba1a38c)

- Configure MySQL server to allow connections from remote hosts.<br>
  `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf` <br>
- Replace ‘127.0.0.1’ to ‘0.0.0.0’ like this:

  ![2_replace_bind_address_config_file](https://github.com/ifydevops23/Application_Architecture/assets/126971054/bb6fe2de-af40-44c6-8e73-02eafafa3d3a)

# STEP 2 - CONNECTING FROM THE CLIENT TO SERVER USING MYSQL UTIILITIES
- On mysql client Linux Server install MySQL Client software.

  ![2_mysql_client_install](https://github.com/ifydevops23/Application_Architecture/assets/126971054/6bb39c67-8634-4ce2-97e1-187af7f5662f)

- From **mysql client** Linux Server connect remotely to mysql server Database Engine without using SSH. <br>
  Type: `mysql -h <PublicIP> -u <my_db_user> -p`

  ![connect_to_db_server_from_client](https://github.com/ifydevops23/Application_Architecture/assets/126971054/f992e4d5-9265-4dce-9334-4f9a188c30f1)

- Check that you have successfully connected to a remote MySQL server and can perform SQL queries:
`SHOW DATABASES;`.
![query_db_from_server](https://github.com/ifydevops23/Application_Architecture/assets/126971054/81956f4f-1913-4485-a33a-1dca696ad248)

- You may create tables also from the client.
 


