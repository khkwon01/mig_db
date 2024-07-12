## MySQL Data Dump(source,aws) and Load (Heatwave MySQL)
### 1. Data Migration Architecture using MySQLShell
![image](https://github.com/user-attachments/assets/7e396423-2c53-4355-853f-df86e4ea7c10)

### 2. Pre-install (if you use oci object storage regarding source data dump storage)
1. install mysql shell on vm (vm is redhat series like oracle, centos etc and it have public ip)    
   yum install mysql-shell
2. oracle cloud command tool installation (based on oracle linux8)
   ```
   dnf -y install oraclelinux-developer-release-el8
   dnf install python36-oci-cli
   ```
   - .oci/config configuration under user account home directory.
   execute the below command if you use oci command on server.  
   ```
   oci-metadata -g region   # check region
   oci setup config         

   # if you execute above command, generated .oci/config files and you can see the following like 
   [DEFAULT]
   user=ocid1.user.oc1..xxxxxx
   fingerprint=xx:xx:xx:xx:xx
   key_file=/root/.oci/oci_api_key.pem
   tenancy=ocid1.tenancy.oc1..xxxxxxxx
   region=ap-chuncheon-1
   ```
3. target db creation (in oci)
     
### 2. vm 기반으로 사용자 데이터 dump / load 
1. source database에서 데이터 dump
   - source database 접속(mysqlsh admin@<<source_ip>>, password: Welcome#1)하여 아래 명령어 수행
     ```
     # 테이블에 pk가 없을 경우 ignore_missing_pks 또는 create_invisible_pks를 compatibility에 추가
     util.dumpSchemas(["airportdb"], "/tmp/airport09", {ocimds: true, threads: 10, showProgress: true, compatibility: ["strip_definers", "strip_restricted_grants"]})  
     ```
2. target database에서 데이터 load  
   - target database 접속(mysqlsh admin@<<source_ip>>, password: Welcome#1)하여 아래 명령어 수행
     ```
     util.loadDump("/tmp/airport09",{threads: 10, updateGtidSet:"append", ignoreExistingObjects: true, resetProgress:true, ignoreVersion:true})
     SELECT @@global.gtid_executed, @@global.gtid_purged;
     ```
3. 이관후 필요하면 replication 연결 ([mds replication연결](https://github.com/khkwon01/mig_db/blob/main/handon/mds_replication_handon.md))
   * *데이터 재이관시 replication 재구성 할 때 아래 명령어로 target 데이터베이스에 gtid 수정을 시도해 보고 안되면 target 데이터베이스 재생성후 replication 구성필요*    
     call sys.set_gtid_purged("+<<소스GTID>:<<GAP_NUM>>");    
     ex) call sys.set_gtid_purged("+f8c1a38e-4ba6-11ee-af6f-0200170028ab:69-111");
     * mysqlshell를 사용하여 데이터 dump시 백업관련 정보는 @.json파일에 저장되어 있어 해당 정보중 gtidExecuted를    
       위 set_gtid_purged에 설정하여 replication 데이터 연결에 사용
       ![image](https://github.com/khkwon01/mig_db/assets/8789421/447d8d42-1245-4ac0-8536-48abcbcd1f94)

4. (참고) 네트웍으로 데이터이관 (MySQL Shell 8.1, 8.0에는 없는 기능임)
   - MySQL Shell에 새로 추가된 기능은 copyinstance, copyschemas를 사용하여 네트웍으로 데이터 이관
     ```
     # 아래는 예제임 (employees schema를 target인 10.1.10.10 mysql 서버에 이관)
     util.copySchemas(['employees'], 'admin@10.1.10.10', {dryRun:false, threads:8, ignoreVersion:true,compatibility: ["strip_definers"]})
     ```
   - 위에 예제로 이관후 결과화면   
     아래 binlog 정보를 사용하여 channel이나 replication를 구성하면 됨.
     ![image](https://github.com/khkwon01/mig_db/assets/8789421/ea94f478-1c45-46a9-8674-c96ff9765997)


### 3. object storage 기반으로 사용자 데이터 dump / load
1. source database에서 데이터 dump
   - source database 접속(mysqlsh admin@<<source_ip>>, password: Welcome#1)하여 아래 명령어 수행
     ```
     util.dumpSchemas(["airportdb"], "airport_dump", {osBucketName:"migdata", osNamespace:"idazzj~~~~", threads:10, ocimds: true})
     ```
2. target database에서 데이터 load   
  ~~ 아래 Case1 또는 Case2를 선책하여 데이터 load 진행 ~~
  - Case 1 : Cloud console에서 load 진행
    * HeatWave MySQL 서비스 생성 항목중에 advanced option를 선택(아래그림)하여 object storage 항목 설정   
      ![image](https://github.com/khkwon01/mig_db/assets/8789421/6e4fa247-19ad-4c86-9d20-67373d6d9007)
    * 기존 Object Storage에서 백업 받은 bucket를 선택하여 load
      ![image](https://github.com/khkwon01/mig_db/assets/8789421/c907a811-726f-4fe0-9b54-0d6a60faba75)

  - Case 2 : target database 접속(mysqlsh admin@<<source_ip>>, password: Welcome#1)하여 아래 명령어 수행
    ```
    util.loadDump("airport_dump", {schema: "airportdb", osBucketName:"migdata", osNamespace:"idazzj~~~~", threads:10})
    ``` 
3. 이관후 필요하면 replication 연결 ([mds replication연결](https://github.com/khkwon01/mig_db/blob/main/handon/mds_replication_handon.md))
