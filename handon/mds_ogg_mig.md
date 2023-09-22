## This document is guide for migrating the data between mds services using ogg.
## 1. Golden gate architecture
![image](https://github.com/khkwon01/mig_db/assets/8789421/6b0be278-0f21-4729-ba1d-0df28c9b5739)

## 2. How to setup the ogg for MySQL HeatWave
### The setup order is the following like   
create two MDS (one is source, the other is target) --> create goldengate deployments -->  
create mds connection in golengate --> login web portal of ogg using admin --> replication setup

### 1) create MySQL HeatWave database (a.k.a mds)
![image](https://github.com/khkwon01/mig_db/assets/8789421/b303e005-12f5-46cd-a0f2-0d8ca1d0809d)

### 2) create goldengate deployments
![image](https://github.com/khkwon01/mig_db/assets/8789421/7bf8ef46-32d3-48ce-b0d0-6ae319a51d02)
- database type : MySQL

### 3) create a connection of mds each server in goldengate
![image](https://github.com/khkwon01/mig_db/assets/8789421/9a6102a0-bc17-4b45-966f-77821bf832cb)

### 4) login ogg web portal using admin
![image](https://github.com/khkwon01/mig_db/assets/8789421/f003fa5c-0f7f-40bd-8255-7f273681d91a)

### 5) setup replication configuration between source and target database    
It needs to be loaded with initial data for service and then you can create the replications regarding changed data.    

If i explain the below capture screen, the extracts is the process which extract the data from source database and the replicats is the process which reflect the data to target database which received the data from source database.   
( cautions : current ogg only apply the DML syntax and data so if you need to apply DDL, you can create the objects in both databases )    


![image](https://github.com/khkwon01/mig_db/assets/8789421/c3a5ccc1-14e5-4988-a62a-cca064bd26a4)

  
