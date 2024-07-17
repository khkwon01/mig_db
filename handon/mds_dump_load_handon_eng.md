## MySQL Data Dump(source,aws) and Load (Heatwave MySQL)
### 1. Data Migration Architecture using MySQLShell
![image](https://github.com/user-attachments/assets/7e396423-2c53-4355-853f-df86e4ea7c10)

### 2. Pre-install (if you use oci object storage regarding source data dump storage)
1. install mysql shell on vm (vm is redhat series like oracle, centos etc and it have public ip)    
   yum install mysql-shell
2. oracle cloud command tool installation (based on oracle linux8)
   ```
   dnf -y install oraclelinux-developer-release-el8
   dnf install python39
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
     
### 2. user data dump / load based on VM 
1. user data dump in source database
   - source database connect (mysqlsh admin@<<source_ip>>, password:<<source password>>) and execute below command
     ```
     # if the table don't have pk, you can add option like `ignore_missing_pks` or `create_invisible_pks`
     util.dumpSchemas(["airportdb"], "/tmp/airport09", {ocimds: true, threads: 10, showProgress: true, compatibility: ["strip_definers", "strip_restricted_grants"]})  
     ```
2. user data load in target database  
   - target database connect (mysqlsh admin@<<source_ip>>, password: Welcome#1) and execute below command
     ```
     util.loadDump("/tmp/airport09",{threads: 10, resetProgress:true, ignoreVersion:true})
     SELECT @@global.gtid_executed, @@global.gtid_purged;
     ```

### 3. user data dump / load based on object storage
1. prerequisite
   - create the bucket for using data dump in oci object storage
3. user data dump in source database
   - source database connect(mysqlsh admin@<<source_ip>>, password:<<source password>>) and execute below command
   - For protecting unnecessory schemas, you can include includesSchemas options
     ```
     util.dumpInstance('test', {osBucketName:"mysql-poc-bucket",osNamespace:"ax5ppfxe6bxg", "ocimds": "true", "includeSchemas": ["",""], threads:5, "compatibility": ["strip_definers", "strip_restricted_grants"]})
     ```
4. user data load in target database 
   - target database connect (mysqlsh admin@<<source_ip>>, password: Welcome#1) and execute below command
    ```
    util.loadDump("test", {osBucketName:"mysql-poc-bucket", osNamespace:"ax5ppfxe6bxg", threads:5})
    ``` 
3. if you need to async data replication, refer to this ([mds replication연결](https://github.com/khkwon01/mig_db/blob/main/handon/mds_replication_handon.md))


### 4. user data dump / load based on Network
1. prerequisite
   - install the mysql shell verion 8.1.0 and use over 8.1.0 version
2. user data dump and load in source database
   - use copyinstance, copyschemas api added new features in 8.1.0
     ```
     # like below example, you connect source database and then execute the following command like (10.1.10.10 is target database)
     util.copySchemas(['employees'], 'admin@10.1.10.10', {dryRun:false, threads:8, ignoreVersion:true,compatibility: ["strip_definers"]})
     ```
   - After migrating the data from source to target, you can get the following info, and then you can use this info when configuring the replication   
     ![image](https://github.com/khkwon01/mig_db/assets/8789421/ea94f478-1c45-46a9-8674-c96ff9765997)

