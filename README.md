# mig_db
migration test for several dbs
## 1. Naver Cloud
### 1. Migration test from Naver MySQL to OCI MDS.   
db간 replication 연결 
네이버 MySQL(8.0.25)에 replictaion을 연결하여 MDS(8.0.33)로 데이터 동기화   
(=> 네이버 MySQL 5.7.29 버전에 대해서도 동작 하는 걸 확인함)
1. MDS replication channel 구성 : (replication fiter : test database)
![image](https://github.com/khkwon01/mig_db/assets/8789421/32acd5af-e255-4c1b-82a9-1ad837fc3bfe)
2. 네이버 MySQL processlist 결과
*![image](https://github.com/khkwon01/mig_db/assets/8789421/fa72b375-2a2c-4ac2-867f-2974670dc143)
3. OCI MDS slave status 결과
![image](https://github.com/khkwon01/mig_db/assets/8789421/364201c4-da1a-4262-ae3e-e129f74ae4e3)
4. 네이버 MySQL에서 데이터 변경
![image](https://github.com/khkwon01/mig_db/assets/8789421/1e02e262-c335-466a-aed0-809e312c5287)
5. OCI MDS에서 데이터 확인
![image](https://github.com/khkwon01/mig_db/assets/8789421/c1f53a8a-5cef-4202-bd0f-379549109297)
   
### 2. Migration test from Naver Server (MariaDB 10.2.11) to OCI MDS
1. Naver 서버(vm) 설정
![image](https://github.com/khkwon01/mig_db/assets/8789421/f47aaef2-6d8f-43b0-bf85-a063572a446a)
    
2. Naver VM내 MySQL 확인
![image](https://github.com/khkwon01/mig_db/assets/8789421/b207a0f2-0138-4679-8115-ede2e7fdf16a)
MySQL Port 오픈 상태 확인
![image](https://github.com/khkwon01/mig_db/assets/8789421/982f75b6-895f-4ee2-94c4-0cbb76c8f0f4)

3. OCI MDS replication 설정   
<span style="color:yellow">노란 글씨입니다.</span>    
5.1 MDS channel replication 설정
![image](https://github.com/khkwon01/mig_db/assets/8789421/bb8ea940-ff1a-4359-8170-dec601b65713)   
3.2 설정후 연결 상태 확인
![image](https://github.com/khkwon01/mig_db/assets/8789421/53fb6783-3ea1-4d43-b94f-caceca095db0)
3.3 MDS replication 상태 확인
![image](https://github.com/khkwon01/mig_db/assets/8789421/22e3fbd1-1dfc-4dc6-91f0-54257c8f703f)
3.4 Naver VM MySQL내 replica 연결 상태
![image](https://github.com/khkwon01/mig_db/assets/8789421/17ee2034-652a-4ce9-a429-f383db238b36)
3.5 Source(Master)에서 데이터 변경시 Slave(Replica)에서 오류발생    
'Worker 1 failed executing transaction 'NOT_YET_DETERMINED' at source log mariadb-bin.000004, end_log_pos 473; Error '@@SESSION.GTID_NEXT cannot be set to ANONYMOUS when @@GLOBAL.GTID_MODE = ON.' on query. Default database: 'test'. Query: 'create database test''    

