
# Configure the Master Server

By default, MySQL server is configured to listen on localhost. So you will need to configure it to listen on the private IP address, set a unique ID and enable the binary logging. You can do it by editing the file depending on the Installation Instance:

 - **1.** Mysql Instance. 
 nano  /etc/mysql/mysql.conf.d/mysqld.cnf

 - **2.** Mariadb Instance
 nano  /etc/mysql/mariadb.conf.d/50-server.cnf


# Steps:


1. Open mysql Configuration File by executing the command below

		nano  /etc/mysql/mariadb.conf.d/50-server.cnf

2. On the Master node, scroll and locate the bind-address attribute as shown below.
		bind-address 	 =127.0.0.1

3. Change the loopback address to match the IP address of the Master node. i.e. The Static IP Address of the Master
		bind-address  	=10.12.0.28

4. Next, specify a value for the server-id attribute in the [mysqld] section. The number you choose should not match any other server-id number. Let’s assign the value 1.
		server-id	 =1
5. At the very end of the configuration file, copy and paste the lines below.

	log_bin = /var/log/mysql/mysql-bin.log
	log_bin_index =/var/log/mysql/mysql-bin.log.index
	relay_log = /var/log/mysql/mysql-relay-bin
	relay_log_index = /var/log/mysql/mysql-relay-bin.index

6. Adding Databases to be Replicated and to be ignored / I solated from being replicated. i.e the Source Databases.

	log-bin                 = mysql-bin
	log-basename            = master1
	binlog_do_db            = reptest
	binlog_ignore_db        = theredcap


7. Save and Exit the configuration file and restart MySQL service for the changes to take effect on Master node.
	
	sudo systemctl restart mysql

8. To verify that MySQL server is running as expected, issue the command.

	sudo systemctl status mysql

	Perfect! MySQL server is running as expected!

9. Create a New User for Replication on Master Node: In this section, we are going to create a replication user in the master node. To achieve this, log in to the MySQL server as shown.
	
	sudo mysql -u root -p

10. Next, proceed and execute the queries below to create a replica user and grant access to the replication slave. Remember to use your IP address. i.e. the replication slave server IP Address

		MariaDB [(none)]> CREATE USER 'replication_user'@'%' IDENTIFIED BY 'myssssssbigs3crets';
		MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO replication_user;
		MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'replication_user'@'10.12.0.29' IDENTIFIED BY 'myssssssbigs3crets' WITH GRANT OPTION;;

11. Next, run the following command to display the master configuration status.
	MariaDB [(none)]> SHOW MASTER STATUS\G;

	The output should be similar to what you can see below.

			MariaDB [(none)]> show master status;
			+--------------------+----------+----------------+------------------+
			| File               | Position | Binlog_Do_DB   | Binlog_Ignore_DB |
			+--------------------+----------+----------------+------------------+
			| master1-bin.000009 |      344 | reptest,redcap |                  |
			+--------------------+----------+----------------+------------------+
			1 row in set (0.000 sec)


**NB:** Be keen and note the master1-bin.000002 value and the Position ID 26685. These values will be crucial when setting up the slave server.
