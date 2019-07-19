# part1


## 1. IP 세팅

```
nd1  52.79.95.134   172.31.39.168
nd1  54.180.14.25   172.31.33.77
nd1  54.180.164.246   172.31.36.182
nd1  54.180.167.90   172.31.44.121
nd1  54.180.170.107   172.31.44.125

All node hosts 파일을 수정한다  --> 프라이빗  IP로 수정 

  $ sudo vi /etc/hosts
172.31.39.168	    nd1.jdh.com nd1
172.31.33.77	    nd2.jdh.com nd2
172.31.36.182       nd3.jdh.com nd3
172.31.44.121	    nd4.jdh.com nd4
172.31.44.125	    nd5.jdh.com nd5
```

## 2. centos pw를 admin 으로 변경한다

```
    $ sudo passwd centos
```


## 3. training user 를 생성하고 권한 준다

```
   group 을 만들고 권한 준다

	$ sudo useradd training -u 3800
	$ sudo passwd training
	$ sudo groupadd skcc
	$ sudo usermod -a -G skcc training
```

## 4. password 정책을 수정한다
```   
   $ sudo vi /etc/ssh/sshd_config
    변경 -> PasswordAuthentication yes
```

## 5. All node sshd.service 재기동
```  
   $ sudo service sshd restart
```

## 6-1 각  node의  hostname 을 바꾼다

```
  $ sudo hostnamectl set-hostname nd1.jdh.com
  $ sudo hostnamectl set-hostname nd2.jdh.com
  $ sudo hostnamectl set-hostname nd3.jdh.com
  $ sudo hostnamectl set-hostname nd4.jdh.com
  $ sudo hostnamectl set-hostname nd5.jdh.com
  
  $ sudo reboot  --> reboot 한다
  
  $ hostname  --> hostname 이 잘 바뀌었는지 확인
```
![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/hostname_nd1.PNG)

## 6-2 skcc 그룹에 training 계정이 들어 갔는지 확인
```
  $ getent group skcc          
  $ getent passwd training
```
![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/getent%20group%20skcc.PNG)

## 6-3 node filesystem 확인
```
  $ df -h
```
![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/nd1%20filesystem.PNG)


## 7. All node CM repo파일 가져오기
```
  $ sudo yum install -y wget
  $ sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d
```

## 8. All node repo 파일 수정

```
  $ sudo vi /etc/yum.repos.d/cloudera-manager.repo
  
  baseurl=https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.15.2/
```

## 9. All node java 1.8 설치
```
  $ sudo yum install java-1.8.0-openjdk-devel.x86_64
```  
  
## 10. All node mysql connector 설치
```
  $ wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
  $ tar zxvf mysql-connector-java-5.1.47.tar.gz
  $ sudo mkdir -p /usr/share/java/
  $ cd mysql-connector-java-5.1.47
  $ sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar  
```  
  
## 11. CM서버 nd1에 CM 설치
```
  $ sudo yum install -y cloudera-manager-daemons cloudera-manager-server
```  

## 12-1. CM서버 nd1에 maria DB 설치
```
  $ sudo yum install -y mariadb-server
  $ sudo systemctl enable mariadb
  $ sudo systemctl start mariadb
  $ sudo /usr/bin/mysql_secure_installation
  
  Enter current password for root (enter for none):
OK, successfully used password, moving on...
[...]
Set root password? [Y/n] Y
New password:
Re-enter new password:
[...]
Remove anonymous users? [Y/n] Y
[...]
Disallow root login remotely? [Y/n] N
[...]
Remove test database and access to it [Y/n] Y
[...]
Reload privilege tables now? [Y/n] Y
[...]
All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.
```

## 12-2. maria DB 접속 후 계정 생성
```
mysql -u root -p --> db 접속
 
  CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

  GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'password';
  GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'password';

  SHOW DATABASES;
 
  EXIT;

nd1에  training 계정 생성
# create user 'training'@'%' identified by 'training';
  GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'training';
  FLUSH PRIVILEGES;
  EXIT;
```
![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/mariaDB_1.PNG)


## 13-1. cloudera manager 설치 (CM Host)
```
sudo yum install cloudera-manager-daemons cloudera-manager-server
```

## 13-2. cloudera manager db 설정 (CM Host)
```
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm password
db.mgmt.properries 존재한다면
sudo rm /etc/cloudera-scm-server/db.mgmt.properties
```

## 13-3. cloudera manager 실행 (CM Host)
```
sudo systemctl start cloudera-scm-server
sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log --> 로그 확인
```
![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_start.PNG)

 
## 14. CM login 후 install

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_Login.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_install_1.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_install_2.PNG)
 
![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_install_3.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_install_4.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_install_5_1.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_install_5_2.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_install_6.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_install_7.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_install_8.PNG)
 
## CM install 후 sqoop 설치

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_sqoop_install.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/CM_sqoop_install_2.PNG)


# Q2
'''
In MySQL create the sample tables that will be used for the rest of the test
a. In MySQL, create a database and name it “test”
b. Create 2 tables in the test databases: authors and posts.
i. You will use the authors.sql and posts.sql script files that will be provided
for you to generate the necessary tables
c. Create and grant user “training” with password “training” full access to the test
database. (It is ok if you give training full access to the entire MySQL database)
'''

## 1) test database 생성

```
 $ mysql -u root -p
 
CREATE DATABASE test;
EXIT;
```

## 2) winscp툴로 쿼리 파일 서버로 전송

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/sql%20file%20load.PNG)

## 3) data import (테이블 및 데이터 생성)
```
  $ mysql -u root -p test < ./authors.sql
  $ mysql -u root -p test < ./posts.sql
```
![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/post%20author%20sql%20%EC%88%98%ED%96%89.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/2%EA%B0%9C%20table%20%EC%83%9D%EC%84%B1.PNG)


## 4) training 계정에 권한 부여
```
  $ mysql -u root -p
GRANT ALL ON test.* TO 'training'@'%' IDENTIFIED BY 'training';
GRANT ALL ON test.* TO 'training'@'localhost' IDENTIFIED BY 'training';
FLUSH PRIVILEGES;
```

## 5) 권한과 table생성, table 별 건 수 결과 확인
```
$ mysql -u training -p

show databases;
use test;
show tables;

desc authors;
desc posts;

select count(*) from authors;
select count(*) from posts;
```

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/%EA%B6%8C%ED%95%9C%EB%B6%80%EC%97%AC%20%EB%B0%8F%20DB%20%ED%99%95%EC%9D%B8.PNG)

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/table%EA%B3%BC%20%EA%B2%B0%EA%B3%BC%ED%99%95%EC%9D%B8.PNG)


# Q3
```
Extract tables authors and posts from the database and create Hive tables.
a. Use Sqoop to import the data from authors and posts
b. For both tables, you will import the data in tab delimited text format
c. The imported data should be saved in training’s HDFS home directory
i. Create authors and posts directories in your HDFS home directory
ii. Save the imported data in each
d. In Hive, create 2 tables: authors and posts. They will contain the data that you
imported from Sqoop in above step.
e. You are free to use whatever database in Hive.
f. Create authors as an external table.
g. Create posts as a managed table.
```

## sqoop로 2개 table import

```
sqoop import --connect jdbc:mysql://172.31.39.168/test --username training --password training --table authors --target-dir /user/training/authors --hive-import --create-hive-table --hive-table default.authors
sqoop import --connect jdbc:mysql://172.31.39.168/test --username training --password training --table posts --target-dir /user/training/posts --hive-import --create-hive-table --hive-table default.posts
```

# Q4
```
Create and run a Hive/Impala query. From the query, generate the results dataset that
you will use in the next step to export in MySQL.
a. Create a query that counts the number of posts each author has created.
i. The id column in authors matches the author_id key in posts.
b. The output of the query should provide the following information:
```

```
select A.id,
       A.first_name AS fname,
       A.last_name AS Lname,
       B.num_posts AS num_posts
	from authors A
	inner join ( select author_id, count(author_id) AS num_posts
                from posts P
               group by author_id ) B
    on A.id = B.author_id
```

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/Q4%20query%20result.PNG)


# Q5
```
Export the data from above query to MySQL
a. Create a MySQL table and name it “results”
i. Make sure it has the necessary columns of matching type as the results of
your query from above
b. The table should be created under the database “test”
c. Finally, export into MySQL the results of your query
```

## results table 생성
```
$ mysql -u training -p

use test;

CREATE TABLE `results` (
  `id` int NOT NULL,
  `fname` varchar(255) default NULL,
  `Lname` varchar(255) default NULL,
  `num_posts` int default 0
);
```

![photo.PNG](https://github.com/nazgoloom/total_test_0719/blob/master/image/results%20table%20creat.PNG)
