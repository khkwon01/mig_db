# mig_db
migration test for several dbs
## 1. Naver Cloud
### 1. Migration test from Naver MySQL to OCI MDS.   
db간 replication 연결 
네이버 MySQL(8.0.25)에 replictaion을 연결하여 MDS(8.0.33)로 데이터 동기화   
(=> 네이버 MySQL 5.7.29 버전에 대해서도 동작 하는 걸 확인함)
* 1) MDS replication channel 구성 : (replication fiter : test database)
![image](https://github.com/khkwon01/mig_db/assets/8789421/32acd5af-e255-4c1b-82a9-1ad837fc3bfe)
* 2) 네이버 MySQL processlist 결과
*![image](https://github.com/khkwon01/mig_db/assets/8789421/fa72b375-2a2c-4ac2-867f-2974670dc143)
* 3) OCI MDS slave status 결과
![image](https://github.com/khkwon01/mig_db/assets/8789421/364201c4-da1a-4262-ae3e-e129f74ae4e3)
* 4) 네이버 MySQL에서 데이터 변경
![image](https://github.com/khkwon01/mig_db/assets/8789421/1e02e262-c335-466a-aed0-809e312c5287)
* 5) OCI MDS에서 데이터 확인
![image](https://github.com/khkwon01/mig_db/assets/8789421/c1f53a8a-5cef-4202-bd0f-379549109297)
   
### 2. Migration test from Naver Server to OCI MDS
1. Naver 서버(vm) 설정
![image](https://github.com/khkwon01/mig_db/assets/8789421/f47aaef2-6d8f-43b0-bf85-a063572a446a)
    
2. Naver VM내 MySQL 확인
![image](https://github.com/khkwon01/mig_db/assets/8789421/b207a0f2-0138-4679-8115-ede2e7fdf16a)
