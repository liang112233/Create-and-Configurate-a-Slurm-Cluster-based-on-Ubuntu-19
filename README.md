# Create-and-Configurate-a-Slurm-Cluster-based-on-Ubuntu-19
#create a slurm cluster with slurmdbd configurated, mysql linked

# step 1: install openssh-server and connect the machines together with ssh

apt-install openssh-server

# step 2: Install NTP and synchronize all machines

#https://vitux.com/how-to-install-ntp-server-and-client-on-ubuntu/

# step 3: Install Munge 


##################This is for all nodes:#############################
sudo su

apt install gcc

apt install openssl

apt install libssl-dev

apt install make

wget https://github.com/dun/munge/archive/munge-0.5.13.tar.gz

tar -zxf munge-0.5.13.tar.gz

cd munge-munge

./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var && make && make install



#####################This is for control node ########################

#create munge key 

dd if=/dev/urandom bs=1 count=1024 >/etc/munge/munge.key

#####################This if for all nodes ###########################

#create user

useradd munge -m -s /bin/bash

#set up passwork

passwd munge

#change owership

chown -R munge.munge /var/{lib,log,run}/munge

chown -R munge.munge /etc/munge


chmod 711 /var/lib/munge

chmod 700 /var/log/munge

chmod 755 /var/run/munge

chmod 700 /etc/munge

chmod 400 /etc/munge/munge.key


##################This is for control node #########################

scp /etc/munge/munge.key munge@ip:/etc/munge

#####################This if for all nodes ###########################

#start munge daemon on all machines 

Start munge:     /etc/init.d/munge start

Check  status:     /etc/init.d/munge status




# step 4: install mysql

#libmysqlclient-dev package is very important, otherwise will cause the slurmdbd can not link to mysql database

apt-get install mysql-server mysql-client libmysqlclient-dev libmysqld-dev

systemctl start mysql.service   # start service  

systemctl status mysql # check status

#set up mysql db

#The following is recommended for /etc/my.cnf, but on CentOS 7 you should create a new file /etc/my.cnf.d/innodb.cnf containing:

[mysqld]

innodb_buffer_pool_size=1024M

innodb_log_file_size=64M

innodb_lock_wait_timeout=900


To implement this change you have to shut down the database and move/remove logfiles:

systemctl stop mysql

mv /var/lib/mysql/ib_logfile? /tmp/

systemctl start mysql

https://wiki.fysik.dtu.dk/niflheim/Slurm_database

#  open mysql in terminal, set mysql

CREATE USER 'slurm'@'localhost IDENTIFIED BY '1234';

GRANT ALL ON slurm_acct_db.* TO 'slurm'@'localhost';

FLUSH PRIVILEGES;

CREATE DATABASE slurm_acct_db;

quit;

mysql -p -u slurm

Tpye password for slurm: 1234. In mysql:

systemctl enable mysql

sacctmgr add cluster fake # fake is my cluster name


# step 5: install slurm

https://www.cnblogs.com/haibaraai0913/p/11045295.html




