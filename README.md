# mig_db
migration test for several dbs

## 1. Migration test from Naver Cloud to OCI MDS.
1. db간 replication 연결
네이버 MySQL(8.0.25)에 replictaion을 연결하여 MDS로 데이터 동기화
* 1) MDS replication channel 구성
![image](https://github.com/khkwon01/mig_db/assets/8789421/9f13c2fb-ab5d-4956-b211-eaba095bf79b)
* 2) 네이버 slave status 결과
![image](https://github.com/khkwon01/mig_db/assets/8789421/364201c4-da1a-4262-ae3e-e129f74ae4e3)
