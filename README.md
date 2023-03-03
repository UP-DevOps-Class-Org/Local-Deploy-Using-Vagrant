

# V-PROFILE WEB PROJECT SETUP

## Prerequisite

 1. Oracle VM Virtualbox
 2. Vagrant
 3. Install Vagrant plugins
	 - vagrant plugin install vagrant-hostmanager   
	 - vagrant plugin install vagrant-vbguest
 4. Git bash or equivalent editor

### VM SETUP
 1. Clone source code.
 2. Cd into the repository.

Bring up vm’s run cmd below

    $ vagrant up

***NOTE***: Bringing up all the vm’s may take a long time based on various factors.
If vm setup stops in the middle run “vagrant up” command again.
***INFO***: All the vm’s hostname and /etc/hosts file entries will be automatically updated


![image](https://user-images.githubusercontent.com/34917417/222372757-2b0ac3f7-01ab-4135-9204-10a9089c0188.png)

### MYSQL Setup

Login to the db vm from terminal

    $ vagrant ssh db01

Verify Hosts entry, if entries missing update the it with IP and hostnames

    # cat /etc/hosts
   
Update OS with latest patches
    
    # yum update -y

Set Repository
    
    # yum install epel-release -y


Install Maria DB Package
    
    # yum install git mariadb-server -y


Starting & enabling mariadb-server

    # systemctl start mariadb
    # systemctl enable mariadb


RUN mysql secure installation script.

    # mysql_secure_installation


***NOTE***: Set db root password, I will be using admin123 as password   

Set DB name and users.

    # mysql -u root -padmin123
    mysql> create database accounts;
    mysql> grant all privileges on accounts.* TO 'admin'@’%’ identified by 'admin123' ;
    mysql> FLUSH PRIVILEGES;
    mysql> exit;

Download Source code & Initialize Database.

    # git clone -b local-setup https://github.com/devopshydclub/vprofile-project.git
    # cd vprofile-project
    # mysql -u root -p admin123 accounts < src/main/resources/db_backup.sql
    # mysql -u root -p admin123 accounts
    mysql> show tables;

Restart mariadb-server
    
    # systemctl restart mariadb

Starting the firewall and allowing the mariadb to access from port no. 3306
    
    # systemctl start firewalld
    # systemctl enable firewalld
    # firewall-cmd --get-active-zones
    # firewall-cmd --zone=public --add-port=3306/tcp --permanent
    # firewall-cmd --reload
    # systemctl restart mariadb

### MEMCACHE SETUP

Install, start & enable memcache on port 11211

    #yum install epel-release -y
    #yum install memcached -y
    #systemctl start memcached
    #systemctl enable memcached
    #systemctl status memcached
    #memcached -p 11211 -U 11111 -u memcached -d

Starting the firewall and allowing the port 11211 to access memcache

    # systemctl enable firewalld
    # systemctl start firewalld
    # systemctl status firewalld
    # firewall-cmd --add-port=11211/tcp --permanent
    # firewall-cmd --reload
    # memcached -p 11211 -U 11111 -u memcache -d
    
### RABBITMQ SETUP

Login to the RabbitMQ vm

    $ vagrant ssh rmq01
    
Verify Hosts entry, if entries missing update the it with IP and hostnames
    
    # cat /etc/hosts    
    
    
Update OS with latest patches and Set EPEL Repository
    
    # yum update -y
    # yum install epel-release -y


Install Dependencies

    # sudo yum install wget -y
    # cd /tmp/
    # wget http://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm
    # sudo rpm -Uvh erlang-solutions-2.0-1.noarch.rpm
    # sudo yum -y install erlang socat
    
    
Install Rabbitmq Server

    # curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
    # sudo yum install rabbitmq-server -y

Start & Enable RabbitMQ

    # sudo systemctl start rabbitmq-server
    # sudo systemctl enable rabbitmq-server
    # sudo systemctl status rabbitmq-server

Restart RabbitMQ service
    # systemctl restart rabbitmq-server
    
Enabling the firewall and allowing port 25672 to access the rabbitmq permanently

    # systemctl start firewalld
    # systemctl enable firewalld
    # firewall-cmd --get-active-zones
    # firewall-cmd --zone=public --add-port=25672/tcp --permanent
    # firewall-cmd --reload

Config Change

    # sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
    # sudo rabbitmqctl add_user test test
    # sudo rabbitmqctl set_user_tags test administrator    
    
    
### TOMCAT SETUP

Login to the tomcat vm
    
    $ vagrant ssh app01
    
Verify Hosts entry, if entries missing update the it with IP and hostnames
    
    # cat /etc/hosts
    
Update OS with latest patches
    
    # yum update -y
    
Set Repository
    # yum install epel-release -y

Install Dependencies

    # yum install java-1.8.0-openjdk -y
    # yum install git maven wget -y

Change dir to /tmp
    
    # cd /tmp/

Download & Tomcat Package

    # wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.tar.gz
    # tar xzvf apache-tomcat-8.5.37.tar.gz

Add tomcat user
    
    # useradd --home-dir /usr/local/tomcat8 --shell /sbin/nologin tomcat

Copy data to tomcat home dir
    
    # cp -r /tmp/apache-tomcat-8.5.37/* /usr/local/tomcat8/

Make tomcat user owner of tomcat home dir
    
    # chown -R tomcat.tomcat /usr/local/tomcat8
   
   