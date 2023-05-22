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
* GTID가 설정이 안된 Mariadb 이관시에는 아래(3.1 ~ 3.5)와 같이 오류가 발생하기 때문에 중간에 gtid로 설정된 
replication 서버 1대가 더 필요함.    
(MDS는 현재 시점 기준 gtid_mode를 ON_PERMISSIVE로 변경 할 수가 없음)
> 3.1 MDS channel replication 설정
![image](https://github.com/khkwon01/mig_db/assets/8789421/bb8ea940-ff1a-4359-8170-dec601b65713)   
> 3.2 설정후 연결 상태 확인
![image](https://github.com/khkwon01/mig_db/assets/8789421/53fb6783-3ea1-4d43-b94f-caceca095db0)
> 3.3 MDS replication 상태 확인
![image](https://github.com/khkwon01/mig_db/assets/8789421/22e3fbd1-1dfc-4dc6-91f0-54257c8f703f)
> 3.4 Naver VM MySQL내 replica 연결 상태
![image](https://github.com/khkwon01/mig_db/assets/8789421/17ee2034-652a-4ce9-a429-f383db238b36)
> 3.5 Source(Master)에서 데이터 변경시 Slave(Replica)에서 오류발생    
'Worker 1 failed executing transaction 'NOT_YET_DETERMINED' at source log mariadb-bin.000004, end_log_pos 473; Error '@@SESSION.GTID_NEXT cannot be set to ANONYMOUS when @@GLOBAL.GTID_MODE = ON.' on query. Default database: 'test'. Query: 'create database test''    
---    
* GTID를 사용하든 안하든 Mariadb는 MySQL과 replication 구조가 틀리기 때문에 아래와 같이 변경하여 구성     
![image](https://github.com/khkwon01/mig_db/assets/8789421/67344261-6e19-4f6e-b8c7-daaf99e5246d)   

3.6 MariaDB : binlog position (default 설정)   
3.7 MySQL 5.7 : binlog position (log-bin.000001/154)    
Naver Cloud 서버에서 제공하는 MySQL 5.7 또는 8.0를 설정후 start (8.0 파라미터는 틀릴수 있음)     
![image](https://github.com/khkwon01/mig_db/assets/8789421/11ad1207-b335-4e10-bbca-458d27224627)    
3.8 MDS 8.0.33 : GTID (default 설정)
* MDS channel replication 설정     
![image](https://github.com/khkwon01/mig_db/assets/8789421/b3adaa65-5193-4af6-af7e-2be6ed7dbc17)
* MDS channel replication 상태
![image](https://github.com/khkwon01/mig_db/assets/8789421/d792fde6-5966-4ffe-8b39-b96542e87d6e)     
3.9 데이터 변경 테스트
* MariaDB에서 데이터 변경     
![image](https://github.com/khkwon01/mig_db/assets/8789421/7f31e90c-e05f-4d33-ae72-20cad00e309b)
* MySQL 5.7 (intermediate stage)
![image](https://github.com/khkwon01/mig_db/assets/8789421/514d4eed-78be-4b74-9277-c69a43e184c2)     
* MDS 8.0.33 (final db)
![image](https://github.com/khkwon01/mig_db/assets/8789421/eeac30ae-b9fd-410c-9e44-195bd6b8997f)



