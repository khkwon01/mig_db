### 1. Prerequesites    
For network between AWS and OCI,   
- use VPN or use Dedicated network (ex,Direct conenct)
- use public ip of RDS

For AWS RDS database,
- enable log-bin and gtid 
- setup binlog_foramt to ROW
- setup binlog_row_image to FULL
- setup binlog_row_metadata to FULL

### 1. Amazon linux setup using MySQL shell
- loing in Amazon linux   
  ```ssh -i "khk-aws-key.pem" ec2-user@<<ec2 vm public ip>>```
- install mysql repo for oracle   
  ```dnf install https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm```
- install mysql-shell   
  ```dnf install mysql-shell```
- login in mysql rds   
  ```mysqlsh admin@database1.xxxxx.ap-northeast-2.rds.amazonaws.com```
- `if you use util.load or util.dump in mysql-shell, you have to enable performance schema paramater modifing the parameter of rds like`
  ![image](https://github.com/user-attachments/assets/2c898dbf-5175-4dcd-ab50-90045e518dfb)
- install oracle oci cli   
  ```bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"```

### 2. Migration Method
#### 1. OCI DMS (Database Migration service)
![image](https://github.com/user-attachments/assets/3c6178b1-a4eb-4072-85d2-618614c5068a)

- Migration Process :  connections create --> migration

- create database connections    
  - aws connection setup    
    ![image](https://github.com/user-attachments/assets/cc6b076f-9a0b-48a6-bdb7-2b532266c75b)   
    ![image](https://github.com/user-attachments/assets/387a4b14-c9b5-4087-9795-02e81044e970)   
  - heatwave connection setup   
    ![image](https://github.com/user-attachments/assets/f09b5c8c-8f0f-45a6-b853-727b2b2c0ae7)
    ![image](https://github.com/user-attachments/assets/4f6a4bc3-1f9e-4ab2-8ef1-a87697f8d4d9)   
  - status check after creating each connection   
    ![image](https://github.com/user-attachments/assets/f38f1121-2aa2-42e3-a400-1833573f4733)   

- create migration and migrate  
  - migration service setup   
    ![image](https://github.com/user-attachments/assets/4e89b5d7-063d-46a9-85d4-bcfbdba4aebe)
    ![image](https://github.com/user-attachments/assets/525c1a1f-fa61-418a-a38b-8d3f9963b750)
    ![image](https://github.com/user-attachments/assets/ff3bf1f0-624d-4ead-9be9-da4ec4449e4a)
    ![image](https://github.com/user-attachments/assets/6ce23834-cc0e-40dc-931c-26b7e59555b2)    
  - status check after creating migration service   
    ![image](https://github.com/user-attachments/assets/48abc3b5-3208-49e5-9a2f-78822b48b4fe)   
  - execute validate for migration (click validate button)   
    ![image](https://github.com/user-attachments/assets/e332a95b-86f8-4b5f-b459-f76fa1356151)   
  - migrate the data from aws to heatwave   
    ![image](https://github.com/user-attachments/assets/1c0a2a85-bce2-48a2-8e8b-69f44643d434)   
    
#### 2. MySQL SHELL (Dump/Load util)
- Prerequesites
  - create object storage in oci and setup oci cli for using object storage in mysql-shell    
    oci env setup : ```oci setup config```
- dump the source data (RDS --> oci object storage)
  ```
  util.dumpInstance('test', {osBucketName:"mysql-test-bucket",osNamespace:"a------xg", "ocimds": "true", "includeSchemas": ["airportdb"], threads:5, "compatibility": ["strip_definers", "strip_restricted_grants", "create_invisible_pks", "force_innodb"]})
  ```
- load the source data into heatwave (oci object storage --> Heatwave)
  ```
  util.loadDump("test", {osBucketName:"mysql-test-bucket", osNamespace:"a------xg", threads:5, resetProgress:true, ignoreVersion:true, updateGtidSet:"append", deferTableIndexes: all})
  ```
- configure channel(replication) if needs
  - https://blogs.oracle.com/mysql/post/successful-rds-to-oci-mysql-heatwave-migration-with-replication-channel-filters

#### 3. 이관 전/후 테이블 비교
```
select md5(sum(aid)), md5(sum(filler)), count(1) from qb_accounts;
```


