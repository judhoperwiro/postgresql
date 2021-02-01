### Install PostgreSQL 12.2 on Ubuntu 18.04.3 LTS or RHEL 7.7

​ 
#### Component
- Ubuntu 18.04.3 LTS for ==Ubuntu==
- RHEL 7.7 for ==Redhat==
- PostgreSQL 12.2
- Prerequisite Package
- postgresql-12.1.tar.gz

``` 
Dynamic Parameter ==***==
Database Instance user: ==<instname>==
Database group name   : ==dbgrp==
Database GID          : ==3000==
Database instance UID : ==<UID>==
PostgreSQL user       : ==<user>==
PostgreSQL password   : ==<pass>==
PostgreSQL port       : ==<dbport>==
db name               : ==<dbname>==
tablespace name       : ==dbspace_<dbname>==
``` 
​ 
## 1.) Install prerequisite library.
Run command as root user.

### 1.1) Prerequisite library for PostgreSQL.
- Install these library.

==For Ubuntu==
``` 
yes Y | sudo apt install gcc
yes Y | sudo apt install libreadline-dev
yes Y | sudo apt install zlib1g-dev
yes Y | sudo apt install make
``` 
==For Redhat==
``` 
yes Y | yum install gcc
yes Y | yum install readline-devel
yes Y | yum install zlib-devel
yes Y | yum install make
``` 
​ 
## 2.) Install PostgreSQL.
Run command as root user.
### 2.1) Create PostgreSQL user
- Create instance owner group.
``` 
groupadd dbgrp -g 3000
``` 
- Create instance owner user and define password.
``` 
useradd <instname> -u <UID> -s /bin/bash -m -d /home/<instname> -g dbgrp
echo -e "<instname>\n<instname>" | passwd <instname>
chage -I -1 -m 0 -M 99999 -E -1 <instname> 
``` 
### 2.2) Extract PostgreSQL source.
- Extract zip file.

==For Ubuntu==
``` 
cd /Source
tar -xvzf postgresql-12.1.tar.gz
chown ubuntu:ubuntu postgresql-12.1
``` 
==For Redhat==
``` 
cd /Source
tar -xvzf postgresql-12.1.tar.gz
chown ec2-user:ec2-user postgresql-12.1
``` 

### 2.3) Prepare PostgreSQL environment.
- Create the database instance directory.
``` 
sudo mkdir -p /data/DB/postgresql/<instname>
sudo chown -R <instname>:dbgrp /data/DB/postgresql/<instname>
``` 
- Specified the environment variable to database instance user:
``` 
echo "" >> /home/<instname>/.bashrc
echo "LD_LIBRARY_PATH=/data/DB/postgresql/<instname>/etc/lib" >> /home/<instname>/.bashrc
echo "PATH=/data/DB/postgresql/<instname>/etc/bin:\$PATH" >> /home/<instname>/.bashrc
echo "PGPORT=<dbport>" >> /home/<instname>/.bashrc
echo "PGDATABASE=<dbname>" >> /home/<instname>/.bashrc
echo "PGUSER=<user>" >> /home/<instname>/.bashrc
echo "PGTZ=Asia/Bangkok" >> /home/<instname>/.bashrc
echo "" >> /home/<instname>/.bashrc
echo "export LD_LIBRARY_PATH" >> /home/<instname>/.bashrc
echo "export PATH" >> /home/<instname>/.bashrc
echo "export PGPORT" >> /home/<instname>/.bashrc
echo "export PGDATABASE" >> /home/<instname>/.bashrc
echo "export PGUSER" >> /home/<instname>/.bashrc
echo "export PGTZ" >> /home/<instname>/.bashrc
echo "" >> /home/<instname>/.bashrc
``` 
- Create additional directory.
``` 
mkdir -p /data/DB/postgresql/<instname>/etc
mkdir -p /data/DB/postgresql/<instname>/db
mkdir -p /data/DB/postgresql/<instname>/log/pg_log
mkdir -p /data/DB/postgresql/<instname>/log/pg_wal
mkdir -p /data/DB/postgresql/<instname>/log/pg_archive

chmod -R 700 /data/DB/postgresql/<instname>/*
chown -R <instname>:dbgrp /data/DB/postgresql/<instname>
``` 

### 2.4) Install PostgreSQL.
- Prepare working directory.

``` 
su - <instname>
cd /Source/postgresql-12.1

mkdir build_dir_<instname>
cd build_dir_<instname>
``` 
- Configure source file.
``` 
../configure --prefix=/data/DB/postgresql/<instname>/etc --with-pgport=<dbport>
``` 
- Compile source file.
``` 
make
``` 
- Install PostgreSQL. After run this command, you must see the result "PostgreSQL installation complete."
``` 
make install
``` 
- Install additional extension.
``` 
cd /Source/postgresql-12.1/build_dir_<instname>/contrib/pg_buffercache
make install

cd /Source/postgresql-12.1/build_dir_<instname>/contrib/adminpack
make install

cd /Source/postgresql-12.1/build_dir_<instname>/contrib/pg_stat_statements
make install

cd /Source/postgresql-12.1/build_dir_<instname>/contrib/pgstattuple
make install

cd /Source/postgresql-12.1/build_dir_<instname>/contrib/file_fdw
make install

exit
``` 
- Create link to PostgreSQL library.
``` 
sudo /sbin/ldconfig /data/DB/postgresql/<instname>/etc/lib
``` 

### 2.5) Initial and configure database cluster.
- Initial database cluster.

``` 
su - <instname>
initdb -D /data/DB/postgresql/<instname>/etc/data
``` 
- Change password authentication policy.
``` 
echo "# Additional option" >> /data/DB/postgresql/<instname>/etc/data/pg_hba.conf

echo "host    all             all             0.0.0.0/0               md5" >> /data/DB/postgresql/<instname>/etc/data/pg_hba.conf

echo "" >> /data/DB/postgresql/<instname>/etc/data/pg_hba.conf

sed -i "s/127.0.0.1\/32            trust/127.0.0.1\/32            md5/g" /data/DB/postgresql/<instname>/etc/data/pg_hba.conf

sed -i "s/\:\:1\/128                 trust/\:\:1\/128                 md5/g" /data/DB/postgresql/<instname>/etc/data/pg_hba.conf

exit
``` 
- Create Database stop/start script.
``` 
sudo vi /etc/init.d/postgres_<instname>
Copy these lines to the script.

  #!/bin/bash
  function stop_postgresql() {
      su <instname> -c '/data/DB/postgresql/<instname>/etc/bin/pg_ctl -D /data/DB/postgresql/<instname>/etc/data stop'
}
function start_postgresql() {
      su <instname> -c '/data/DB/postgresql/<instname>/etc/bin/pg_ctl -D /data/DB/postgresql/<instname>/etc/data -l /data/DB/postgresql/<instname>/log/pg_log/pg_server.log start'
}
function restart_postgresql() {
      su <instname> -c '/data/DB/postgresql/<instname>/etc/bin/pg_ctl -D /data/DB/postgresql/<instname>/etc/data stop'
      sleep 5
      su <instname> -c '/data/DB/postgresql/<instname>/etc/bin/pg_ctl -D /data/DB/postgresql/<instname>/etc/data -l /data/DB/postgresql/<instname>/log/pg_log/pg_server.log start'
}
function status_postgresql() {
      su <instname> -c '/data/DB/postgresql/<instname>/etc/bin/pg_ctl -D /data/DB/postgresql/<instname>/etc/data status'
}
function reload_postgresql() {
      su <instname> -c '/data/DB/postgresql/<instname>/etc/bin/pg_ctl -D /data/DB/postgresql/<instname>/etc/data reload'
}
case $1 in
  start)
      start_postgresql
      exit $?
      ;;
  stop)
     stop_postgresql
      exit $?
      ;;
  status)
      status_postgresql
      exit $?
      ;;
  restart)
     restart_postgresql
      exit $?
      ;;
  reload)
      restart_postgresql
      exit $?
      ;;
  *)
      echo "Usage: $0 {start|stop|restart|status|reload}"
      exit 2
esac
exit 0
``` 
- change script permission.
``` 
sudo chown <instname>:dbgrp /etc/init.d/postgres_<instname>
sudo chmod 744 /etc/init.d/postgres_<instname>
``` 
### 2.6) Parameter tuning.
- Backup database configuration.
``` 
su - <instname>

cp -p /data/DB/postgresql/<instname>/etc/data/postgresql.conf /data/DB/postgresql/<instname>/etc/data/postgresql.conf.$(date +%Y%m%d)
``` 

- Allow any incoming IP address to access PostgreSQL cluster.
``` 
sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Database Max connection.
``` 
sed -i "s/max_connections = 100/max_connections = 500/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Connection slots that are reserved for connections by PostgreSQL superusers.
``` 
sed -i "s/#superuser_reserved_connections = 3/superuser_reserved_connections = 10/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Memory that dedicated to PostgreSQL to use for caching data (for 6 GB memory).
``` 
sed -i "s/shared_buffers = 128MB/shared_buffers = 1536MB/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specify this parameter to allows PostgreSQL to do larger in-memory sorts (for 6 GB memory).
``` 
sed -i "s/#work_mem = 4MB/work_mem = 3145kB/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the maximum amount of memory to be used by maintenance operations (for 6 GB memory).
``` 
sed -i "s/#maintenance_work_mem = 64MB/maintenance_work_mem = 384MB/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the WAL buffers size (for 6 GB memory).
``` 
sed -i "s/#wal_buffers = -1/wal_buffers = 16MB/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies maximum number of temporary buffers used by each database session (for 6 GB memory).
``` 
sed -i "s/#temp_buffers = 8MB/temp_buffers = 8MB/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the memory that available for disk caching (for 6 GB memory).
``` 
sed -i "s/#effective_cache_size = 4GB/effective_cache_size = 4608MB/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the number of shared libraries to be preloaded at server start (for 6 GB memory).
``` 
sed -i "s/#shared_preload_libraries = ''/shared_preload_libraries = 'pg_stat_statements'/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the minimum WAL file size checkpoint.
``` 
sed -i "s/min_wal_size = 80MB/min_wal_size = 2GB/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the maximum WAL file size checkpoint.
``` 
sed -i "s/max_wal_size = 1GB/max_wal_size = 4GB/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the maximum time between automatic WAL checkpoints.
``` 
sed -i "s/#checkpoint_timeout = 5min/checkpoint_timeout = 5min/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the target of checkpoint completion, as a fraction of total time between checkpoints.
``` 
sed -i "s/#checkpoint_completion_target = 0.5/checkpoint_completion_target = 0.9/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the delay between activity rounds for the background writer.
``` 
sed -i "s/#bgwriter_delay = 200ms/bgwriter_delay = 200ms/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
```  
- Specifies the background writing limits, the default value is 100 buffers.
``` 
sed -i "s/#bgwriter_lru_maxpages = 100/bgwriter_lru_maxpages = 100/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the multiplier of the estimate number of dirty buffers that will be needed during the next round.
``` 
sed -i "s/#bgwriter_lru_multiplier = 2.0/bgwriter_lru_multiplier = 2.0/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the maximum number of background processes that the system can support.
``` 
sed -i "s/#max_worker_processes = 8/max_worker_processes = 3/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the maximum number of autovacuum processes (other than the autovacuum launcher) that may be running at any one time.
``` 
sed -i "s/#autovacuum_max_workers = 3/autovacuum_max_workers = 3/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the default statistics target for table columns without a column-specific target set via ALTER TABLE SET STATISTICS.
``` 
sed -i "s/#default_statistics_target = 100/default_statistics_target = 100/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the planner's estimate of the cost of a non-sequentially-fetched disk page.
``` 
sed -i "s/#random_page_cost = 4.0/random_page_cost = 1.1/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the number of concurrent disk I/O operations that PostgreSQL expects can be executed simultaneously.
``` 
sed -i "s/#effective_io_concurrency = 1/effective_io_concurrency = 200/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Set a printf-style string that is output at the beginning of each log line.
``` 
sed -i "s/#log_line_prefix = '%m \[%p\] '/log_line_prefix = '%m %t %u \[%p\] %x '/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the directory in which log files will be created.
``` 
sed -i "s/#log_directory = 'log'/log_directory = '\/data\/DB\/postgresql\/<instname>\/log\/pg_log'/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf
``` 
- Specifies the following parameters to Enable Archive
``` 
sed -i "s/#max_wal_senders = 10/max_wal_senders = 10/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf

sed -i "s/#wal_level = replica/wal_level = archive/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf

sed -i "s/#archive_mode = off/archive_mode = on/g" /data/DB/postgresql/<instname>/etc/data/postgresql.conf

sed -i "s/#archive_command = ''/archive_command = \'cp -i %p \/data\/DB\/postgresql\/<instname>\/log\/pg_archive\/%f <\/dev\/null \'/g " /data/DB/postgresql/<instname>/etc/data/postgresql.conf

exit
``` 
- Start Database Cluster.
``` 
sudo /etc/init.d/postgres_<instname> start

``` 
### 2.7) Create the user, tablespace, database.
#### 2.7.1) Create PostgreSQL user (Bypass this step if the ==Database Instance user== and ==PostgreSQL user== are the same name)
``` 
su - <instname>
createuser --interactive --pwprompt
when prompt :

Enter name of role to add: ==<user>==
Enter password for new role: ==<pass>==
``` 
#### 2.7.2) Create tablespace.
``` 
su - <instname>
mkdir -p /data/DB/postgresql/<instname>/db/<dbname>
psql -d postgres -c "CREATE TABLESPACE dbspace_<dbname> OWNER <user> LOCATION '/data/DB/postgresql/<instname>/db/<dbname>' "
2.7.3) Create database.

psql -d postgres -c "CREATE DATABASE <dbname> OWNER <user> TABLESPACE dbspace_<dbname>"
``` 
#### 2.7.4) Create users and Set default schema of users.
``` 
su - <instname>
psql -d postgres -c "create user postgres with password 'root';"
``` 
#### 2.7.5) Grant super user privilege.
``` 
su - <instname>
psql -d postgres -c "alter user postgres with superuser;"
``` 
