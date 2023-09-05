## MDS Handon for Data dump or load

### 1. 필요 사항
1. install mysql shell on vm (vm is redhat series like oracle, centos etc and it have public ip)
   yum install mysql-shell
2. sample data : [airport db](https://dev.mysql.com/doc/airportdb/en/)
3. oracle cloud command tool 설치 (oracle linux8 기준)
   ```
   dnf -y install oraclelinux-developer-release-el8
   dnf install python36-oci-cli
   ```
4. source db 및 target db 생성 : [![Deploy to Oracle Cloud](https://oci-resourcemanager-plugin.plugins.oci.oraclecloud.com/latest/deploy-to-oracle-cloud.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=https://github.com/khkwon01/oci-mysql-config/archive/refs/tags/mds-provision-3.7.zip)
5. source 데이터 load
   - 2번 sample 데이터를 vm상에 다운로드  
   - 4번에서 생성된 source database 접속 (mysqlsh admin@<<source_ip>>, password: Welcome#1) 아래 명령어 수행하여 load
     ```   
     util.loadDump("/home/test/airport-db <-- 환경에 따라 변경", {dryRun: false, threads: 8, resetProgress:true, ignoreVersion:true})
     ```
6. source load된 데이터로 CRUD 테스트
     
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
