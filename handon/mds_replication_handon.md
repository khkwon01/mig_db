# 1. 전체 MDS 구성도    
OCI에 있는 MDS, HeatWave, Replication 테스트를 하기 위해 아래와 같이 구성하여 테스트 진행    
Source(MySQL 5.7)를 제외하고는 아래 Delpoy를 아이콘을 클릭하여 설치를 진행하면 자동으로 구성을 진행함      
[![Deploy to Oracle Cloud](https://oci-resourcemanager-plugin.plugins.oci.oraclecloud.com/latest/deploy-to-oracle-cloud.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=https://github.com/khkwon01/terraform-mds/archive/refs/tags/mds-heatwave-v3.3.0.zip)   

![Alt text](image.png)

# 2. Source 구성 (MySQL 5.7)  
위에 그림에서 MySQL 5.7에 해당 하는 부분은 직접 VM 인스턴스를 생성하여 MySQL 설치하여 구성함    
**주의사항** 소스 DB가 MDS일 경우 아래 파라미터를 꼭 길게 설정(1일이상)해서 사용하시기 바랍니다.   
binlog_expire_logs_seconds (default 1시간, 1시간이 지나면 binlog를 지우기 때문에 설정을 변경해서 길게 잡아야 함)    

## 1) VM 구성    
Name과 Oracle Linux8 os, operator-subnet-regional subnet등을 선택하여 설치

기본적으로 firewall은 disable 시켜야 함.    
```
systemctl stop firewalld.service 
systemctl disable firewalld.service 
```
## 2) MySQL 5.7 Download    
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.42-linux-glibc2.12-x86_64.tar

## 3) MySQL 추가 패키지 설치   
yum install -y ncurses-compat-libs

## 4) MySQL 5.7 설치
```
tar -xvf mysql-5.7.42-linux-glibc2.12-x86_64.tar
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
echo "export PATH=$PATH:/usr/local/mysql/bin" >> /root/.bash_profile
. ~/.bash_profile
// 가동 후 임시 패스워드 변경  set password = password('Welcome#1')
set password=password('Welcome#1');
cp support-files/mysql.server /etc/init.d/mysql.server

printf "[mysqld]\ngtid_mode=on\nenforce-gtid-consistency\nlog-bin=/usr/local/mysql/data/logbin\nserver_id=0720\n" >> /etc/my.cnf
/etc/init.d/mysql.server restart
```
## 5) Test 계정 생성 및 데이터 import
```
// 샘플 데이터 다운로드
wget https://downloads.mysql.com/docs/world-db.zip

// 샘플 데이터 import
mysql -u root -p < world.sql

// 테스트 계정 생성
mysql> create user svctest@'%' identified by "Welcome#1";
mysql> grant all privileges on world.* to svctest@'%';

// replication 계정 생성
mysql> create user repl@'%' identified by "Welcome#1";
mysql> grant replication slave on *.* to repl@'%';
```
# 3. Target 구성 (MDS, HeatWave)
## 1) 초기 데이터 적재
```
// 샘플 데이터 import
mysql -u admin -h <<mds or heatwave ip>> -p < world.sql
```
    
// 실제 데이터를 소스에서 dump해서 MDS로 옮길 경우 아래와 같은 절차로 진행 하시기 바랍니다. (mysqlshell)  
```
util.dumpSchemas(["test"], "/tmp/test", {ocimds: true, threads: 4});      // schema 기준 데이터 dump (GTID 포함 dump)

util.loadDump("/tmp/test",{updateGtidSet:"append", ignoreVersion: true})    // 신규일 경우 : 소스에 GTID를 포함하여 타겟에 설정함
util.loadDump("/tmp/test",{ignoreVersion: true})                            // GTID 오류가 날 경우 사용, 그리고 나서  call sys.set_gtid_purged로 조정
```


## 2) channel - replication ( 메뉴 Databases > DB systems > channels ) 
### A. Source connection (공통)   
아래 항목은 channel 구성(전체, 스키마, db, table등)시 기본 공통적으로 설정을 해야 하는 부분임.     
![image](https://github.com/khkwon01/mig_db/assets/8789421/9676817d-78f5-4018-9ac0-3c0520107fa3)

### B. Target DB system - 전체
Source와 Target db간 전체 데이터가 동일해야 함   
(channel filter 참고 자료 : https://docs.oracle.com/en-us/iaas/mysql-database/doc/creating-replication-channel.html#GUID-DF828619-669E-41CC-8BE5-F7A136AFF470)    
- replication 구성 
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/5b98d5dd-3e7a-482d-9a1f-654a1e919f81)

- replication 완료후 상태   
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/22f85991-de63-4ed1-a108-a76659978f38)

- replication 테스트
  - Source DDL/DML 수행   
    ```
    // source에서 아래와 같이 수행하면 target에서 생성
    create database test1;
    use test1;
    create table t1 (id int primary key, nm varchar(10));
    insert into t1 values (1, 'nm1'), (2, 'nm2'), (3, 'nm3');
  
    // target에서 아래 명령어를 수행하면 복제된 걸 확인 가능
    ```
  - Target 조회 결과    
    <img width="823" alt="image" src="https://github.com/khkwon01/mig_db/assets/8789421/33e3b891-87c6-4e26-a2ea-c3a921f16ee8">
   
### C. Target DB system - db   
Source와 Target db간 schema 기준 데이터가 동일해야 함
- replication 구성
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/71d2b80e-00ee-4c54-9f12-36f1af10c572)    
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/39b25f43-11c2-414d-85f6-d623372a4c45)

- replication 완료후 상태    
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/4be4d667-9691-4fdc-afc3-5b92db953051)

- replication 테스트
  - Source DDL/DML 수행   
    ```
    use world;
    create table t1 (id int primary key, nm varchar(10));
    insert into t1 values (1, 'nm1'), (2, 'nm2'), (3, 'nm3');
    update t1 set nm = 'changenm2' where id = 2;
    ```
  - Target 조회 결과    
    <img width="751" alt="image" src="https://github.com/khkwon01/mig_db/assets/8789421/df635af9-3b62-4615-998c-ce49c0c6d4cf">

### D. Target DB system - table-*    
Source와 Target db간 테이블 기준 데이터가 동일해야 함    
- replication 구성
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/42bc905a-d2ea-42f0-8764-9c7300a716db)
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/d866119c-357f-47de-8e42-aaaf37dd07fe)


- replication 완료후 상태    
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/f2c0c80d-4efb-4c96-88a8-de5017dddb5c)

- replication 테스트
  - Source DDL/DML 수행   
    ```
    use world;
    insert into t1 values (4, 'nm4'), (5, 'nm5'), (6, 'nm6');
    update t1 set nm = 'changenm5' where id = 5;
    create table t2 (id int primary key, nm varchar(10));
    insert into t2 values (1, 'nm1'), (2, 'nm2'), (3, 'nm3');
    ```
  - Target 조회 결과     
    <img width="755" alt="image" src="https://github.com/khkwon01/mig_db/assets/8789421/2b5e9a1b-6016-49f7-bf31-5f16ccca09ea">
  
  - 추가 테스트    
    - 1개 정책에 여러개 table 이름 패턴 추가 - 지원안됨     
      ![image](https://github.com/khkwon01/mig_db/assets/8789421/55a77317-f7d3-4ad0-bad3-9b978f5936f8)     
    - 여러개 정책에 table 이름 패턴 추가 - 지원     
      ![image](https://github.com/khkwon01/mig_db/assets/8789421/9184ac5e-32b2-48f3-8c25-db35a59ea972)    

### E. Target DB system - rewrite-db 
Source와 Target db간 db 이름만 다르고 데이터는 동일해야 함
- replication 구성
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/6e921f2d-c319-46ad-a848-7a03b0bda25a)

- replication 완료후 상태      
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/4c7c9f10-b624-499a-ad6b-3ad83a0daede)

- replication 테스트
  - Source DDL/DML 수행   
    ```
    use world;
    update city set name='Kabul_change' where id = 1;    
    create table t1 (id int primary key, nm varchar(10));
    insert into t1 values (1, 'nm1'), (2, 'nm2'), (3, 'nm3');    
    ```
  - Target 조회 결과     
    <img width="889" alt="image" src="https://github.com/khkwon01/mig_db/assets/8789421/dd92f28f-e7a6-45b2-8d79-5c07c136586a"> 
    <img width="863" alt="image" src="https://github.com/khkwon01/mig_db/assets/8789421/48d22449-b07b-4b12-92ad-41d766a07184">

## 3) channel(replication) 로그 확인
### A. source (db접속)
    ```
    SHOW GLOBAL VARIABLES;
    SHOW GLOBAL STATUS;
    SHOW ENGINE INNODB STATUS;
    SHOW MASTER STATUS;
    SHOW MASTER LOGS;
    SHOW BINARY LOG STATUS\G;
    SELECT * FROM performance_schema.error_log;
    ```
### B. target (db접속)
    ```
    SHOW GLOBAL VARIABLES;
    SHOW GLOBAL STATUS;
    SHOW ENGINE INNODB STATUS;
    SHOW REPLICA STATUS;
    SELECT * FROM performance_schema.replication_connection_configuration\G
    SELECT * FROM performance_schema.replication_connection_status\G
    SELECT * FROM performance_schema.replication_applier_status\G
    SELECT * FROM performance_schema.replication_applier_status_by_coordinator\G
    SELECT * FROM performance_schema.replication_applier_status_by_worker\G
    SELECT * FROM performance_schema.error_log;

    # replication 지연 확인
    select * from sys.replication_lag;
    select * from sys.replication_status_full\G
    ```    
## 4) channel 강제로 연결 (소스단에 binlog가 없어졌을 경우)
### A. 소스 DB에서 현 시점 최종 gtid 확인
  ```
    select @@gtid_executed, @@gtid_purged\G    
    예제로 아래와 같이 내용이 출력된다면,  아래 내용중  165bb3ee-3585-11ee-901d-02001701d33a:1-6 를 복사하고    
    *************************** 1. row ***************************    
    @@gtid_executed: 165bb3ee-3585-11ee-901d-02001701d33a:1-6   
    @@gtid_purged: 165bb3ee-3585-11ee-901d-02001701d33a:1-5   
  ```

### B. 타켓 DB에서  admin 계정으로 아래 명령어 수행    
- 위에 소스 DB에서 copy한 gtid를 넣어주고 아래 명령어 수행 (target에서 executed가 4까지 되었을 경우)    
  call sys.set_gtid_purged("+165bb3ee-3585-11ee-901d-02001701d33a:5-6")     

- or
  ```
   SET GTID_NEXT = 'uuid:transaction_number'; 
   Begin; commit;
   ...
   set gtid_next=AUTOMATIC
  ```

### C. channel 재연결 (resume)


## 5) channel(replication) 상태 확인
- SHOW REPLICA STATUS\G 의 출력 ​​(가장 일반적으로 사용되지만 가장 권장되지는 않음)
- 테이블 performance_schema.replication_connection_status (IO_thread가 실행 중이 아닐 때, connection에 문제가 있을 경우)
- performance_schema.replication_applier_status_by_coordinator 및 performance_schema.replication_applier_status_by_worker 테이블에서 (SQL_thread가 실행 중이 아닐 때, 데이터 반영 안될 경우)
- MySQL의 오류 로그 파일은 performance_schema.error_log 테이블에서도 사용 가능 (특히 MySQL HeatWave Database Service에서 클라우드에서 유용함)


참고자료 : https://blogs.oracle.com/mysql/post/mysql-heatwave-database-service-inbound-replication-channel-troubleshooting-guide
