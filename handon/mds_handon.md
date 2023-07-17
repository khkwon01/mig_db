# 1. 전체 MDS 구성도    
OCI에 있는 MDS, HeatWave, Replication 테스트를 하기 위해 아래와 같이 구성하여 테스트 진행    
Source(MySQL 5.7)를 제외하고는 아래 Delpoy를 아이콘을 클릭하여 설치를 진행하면 자동으로 구성을 진행함      
[![Deploy to Oracle Cloud](https://oci-resourcemanager-plugin.plugins.oci.oraclecloud.com/latest/deploy-to-oracle-cloud.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=https://github.com/khkwon01/terraform-mds/archive/refs/tags/mds-heatwave-v2.6.0.zip)   

![Alt text](image.png)

# 2. Source 구성 (MySQL 5.7)  
위에 그림에서 MySQL 5.7에 해당 하는 부분은 직접 VM 인스턴스를 생성하여 MySQL 설치하여 구성함

## 1) VM 구성    
Name과 Oracle Linux8 os, operator-subnet-regional subnet등을 선택하여 설치

## 2) MySQL 5.7 Download    
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.42-linux-glibc2.12-x86_64.tar

## 3) MySQL Shell 설치   
yum install -y mysql-shell  
yum install -y ncurses-compat-libs

## 4) MySQL 5.7 설치
```
tar -xvf mysql-5.7.42-linux-glibc2.12-x86_64.tar.gz
mv mysql-5.7.42-linux-glibc2.12-x86_64 /usr/local/mysql

groupadd mysql
useradd -r -g mysql -s /bin/false mysql
cd /usr/local/mysql
mkdir mysql-files
chown mysql:mysql mysql-files
chmod 750 mysql-files
bin/mysqld --initialize --user=mysql
bin/mysql_ssl_rsa_setup
bin/mysqld_safe --user=mysql &
// 가동 후 임시 패스워드 변경  set password = password('Welcome#1')
echo "export PATH=$PATH:/usr/local/mysql/bin" >> /root/.bash_profile
. ~/.bash_profile
set password=password('Welcome#1');
cp support-files/mysql.server /etc/init.d/mysql.server

printf "[mysqld]\ngtid_mode=on\nenforce-gtid-consistency\nlog-bin=/usr/local/mysql/data/logbin\nserver_id=0720\n" >> /etc/my.cnf
/etc/init.d/mysql.server restart
```
## 5) Test 계정 생성 및 데이터 import
```
// 샘플 데이터 다운로드
wget https://downloads.mysql.com/docs/world-db.zip

// 데이터 import
mysql -u root -p < world.sql

// 테스트 계정 생성
mysql> create user svctest@'%' identified by "Welcome#1";
mysql> grant all privileges on world.* to svctest@'%';

// replication 계정 생성
mysql> create user repl@'%' identified by "Welcome#1";
mysql> grant replication slave on *.* to repl@'%';
```
# 3. Target 구성 (MDS, HeatWave)