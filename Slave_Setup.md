# Configure the MySQL Slave Server

Head out to the slave server and like we did with the Master server, open the MySQL configuration file

By default, MySQL server is configured to listen on localhost. So you will need to configure it to listen on the private IP address, set a unique ID and enable the binary logging. You can do it by editing the file depending on the Installation Instance:

 1. Mysql Instance. 
 			nano  /etc/mysql/mysql.conf.d/mysqld.cnf

 2. Mariadb Instance
 			nano  /etc/mysql/mariadb.conf.d/50-server.cnf

# Steps:

1. Open mysql Configuration File by executing the command below

			nano  /etc/mysql/mariadb.conf.d/50-server.cnf

2. On the Slave node, scroll and locate the bind-address attribute as shown below.
			bind-address 	 =127.0.0.1

3. Change the loopback address to match the IP address of the Slave node. i.e. The Static IP Address of the Slave
			bind-address  	=10.12.0.29

4. As before, specify a value for the server-id attribute in the [mysqld] section. This time select a different value. Let’s go with 2.
			server-id	 =2

5. Again, paste the lines below at the very end of the configuration file.
	
			log_bin = /var/log/mysql/mysql-bin.log
			log_bin_index =/var/log/mysql/mysql-bin.log.index
			relay_log = /var/log/mysql/mysql-relay-bin
			relay_log_index = /var/log/mysql/mysql-relay-bin.index

6. Once done Save and Exit the configuration file and restart MySQL service for the changes to take effect on Slave node.
	
			sudo systemctl restart mysql

7. Next, log in to the MySQL shell as shown.
	
			sudo mysql -u root -p

	In this step, you will need to make some configuration that will allow the slave server to connect to the master server. But first, stop the slave threads as shown.

			MariaDB [(none)]> STOP SLAVE; 

	The output should be similar to what you can see below.
			MariaDB [(none)]> STOP SLAVE; 
			Query OK, 0 rows affected (0.01 sec)


8. To allow the slave server to replicate the Master server, run the command. 

			CHANGE MASTER TO MASTER_HOST='10.12.0.28', MASTER_USER='replication_user', MASTER_PASSWORD= 'myssssssbigs3crets', MASTER_LOG_FILE='master1-bin.000009', MASTER_LOG_POS=344;

	NB: If you are keen enough, you will observe that we’ve used the mysql-bin.00002 value and position ID 1643 earlier displayed after creating the slave replication user. Additionally, the Master server’s IP address, replication user and password have been used. Later, start the thread you had earlier stopped.

			MariaDB [(none)]> START SLAVE; 

	The output should be similar to what you can see below.

			MariaDB [(none)]> START SLAVE; 
			Query OK, 0 rows affected (0.01 sec)


9. Verify the MySQL Master-Slave Replication. Execute the following command.

	MariaDB [(none)]>  show slave status\G;

	The output should be similar to what you can see below.

		MariaDB [(none)]> show slave status\G;
		*************************** 1. row ***************************
		               Slave_IO_State: Waiting for master to send event
		                  Master_Host: 10.0.6.3
		                  Master_User: replication_user
		                  Master_Port: 3306
		                Connect_Retry: 60
		              Master_Log_File: master1-bin.000009
		          Read_Master_Log_Pos: 88416502
		               Relay_Log_File: mysql-relay-bin.000007
		                Relay_Log_Pos: 786814
		        Relay_Master_Log_File: master1-bin.000009
		             Slave_IO_Running: Yes
		            Slave_SQL_Running: Yes

NB: All is set.