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
   - 4번에서 생성된 source database 접속 (mysqlsh admin@<<source ip>>, password: Welcome#1) 아래 명령어 수행하여 load     
     util.loadDump("/home/test/airport-db <-- 환경에 따라 변경", {dryRun: false, threads: 8, resetProgress:true, ignoreVersion:true})
6. source load된 데이터로 CRUD 테스트
     
### 2. vm 기반으로 사용자 데이터 dump / load 
1. source database에서 데이터 dump
   - source database 접속(mysqlsh admin@<<source ip>>, password: Welcome#1)하여 아래 명령어 수행
     util.dumpInstance("/tmp/airport09", {ocimds: true, threads: 10, showProgress: true, compatibility: ["strip_definers", "strip_restricted_grants"]})    
2. target database에서 데이터 load
   - 
