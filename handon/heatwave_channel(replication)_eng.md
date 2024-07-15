## 1. Channle configuration flow    
![image](https://github.com/user-attachments/assets/de4ed208-231f-4e11-8742-deeb945b86a3)

## 2. Limitations and Cautions
refer to the below url for this.
- https://docs.oracle.com/en-us/iaas/mysql-database/doc/limitations2.html
  - only row-based replication is supported
  - only replication from single source is supported
  - Changes to mysql schema are not replicated and can be cause or break replication

## 3. Channel - replication (OCI Menu: Databases > DB systems > channels ) 
- In channel configuraiton, there are multi-section for configuring the replication between source and target
  ### A. Source connection (common)    
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/9676817d-78f5-4018-9ac0-3c0520107fa3)    

  ### B. Target DB system - filter setup for AWS RDS
  The Aws RDS included extra schema and table for their service which it is not officials.
  So firstly, you must have to filter that schema and tables. (There are several option according to database type)       
  - You click "Show channel filter options" in set-up information    
    ![image](https://github.com/user-attachments/assets/7482811a-36f2-4d1b-b2b2-c0bc94263c76)    
  - If you see channel filter section, you can filter AWS RDS schema and tables like this     
    ![image](https://github.com/user-attachments/assets/585f8ae5-a946-40f2-aedd-933efaf6ce5a)

  And if you need to add some filter for your data, you can filter the next course.

  ### C. Target DB system - replication for all schema
  Must be same data between source and target through dump data  
  (channel filter referenace manual : https://docs.oracle.com/en-us/iaas/mysql-database/doc/creating-replication-channel.html#GUID-DF828619-669E-41CC-8BE5-F7A136AFF470)    
  - channel(replicatio) configuration
    ![image](https://github.com/khkwon01/mig_db/assets/8789421/5b98d5dd-3e7a-482d-9a1f-654a1e919f81)

  - check channel status after configuring replication (green is normal status)   
    ![image](https://github.com/khkwon01/mig_db/assets/8789421/22f85991-de63-4ed1-a108-a76659978f38)
  
  ### D. Target DB system - replication for only specific db(schema)   

  - channel(replicatio) configuration
    ![image](https://github.com/khkwon01/mig_db/assets/8789421/71d2b80e-00ee-4c54-9f12-36f1af10c572)    
    ![image](https://github.com/khkwon01/mig_db/assets/8789421/39b25f43-11c2-414d-85f6-d623372a4c45)

  - check channel status after configuring replication (green is normal status)     
    ![image](https://github.com/khkwon01/mig_db/assets/8789421/4be4d667-9691-4fdc-afc3-5b92db953051)

  ### E. Target DB system - replication for table-*      
  - channel(replicatio) configuration
    ![image](https://github.com/khkwon01/mig_db/assets/8789421/42bc905a-d2ea-42f0-8764-9c7300a716db)
    ![image](https://github.com/khkwon01/mig_db/assets/8789421/d866119c-357f-47de-8e42-aaaf37dd07fe)
    
  - check channel status after configuring replication (green is normal status)     
  ![image](https://github.com/khkwon01/mig_db/assets/8789421/f2c0c80d-4efb-4c96-88a8-de5017dddb5c)  

  ### F. Target DB system - replication for rewriting source db name(schema)
  - channel(replicatio) configuration
    ![image](https://github.com/khkwon01/mig_db/assets/8789421/6e921f2d-c319-46ad-a848-7a03b0bda25a)
  
  - check channel status after configuring replication (green is normal status)        
    ![image](https://github.com/khkwon01/mig_db/assets/8789421/4c7c9f10-b624-499a-ad6b-3ad83a0daede)

## 3) channel(replication) log check(mornitoring)
### A. source (db connection)
    ```
    SHOW GLOBAL VARIABLES;
    SHOW GLOBAL STATUS;
    SHOW ENGINE INNODB STATUS;
    SHOW MASTER STATUS;
    SHOW MASTER LOGS;
    SELECT * FROM performance_schema.error_log;
    ```
### B. target (db connection)
    ```
    SHOW GLOBAL VARIABLES;
    SHOW GLOBAL STATUS;
    SHOW ENGINE INNODB STATUS;
    SHOW REPLICA STATUS;
    SELECT * FROM performance_schema.replication_connection_configuration\G
    SELECT * FROM performance_schema.replication_connection_status\G
    SELECT * FROM performance_schema.replication_applier_status\G
    SELECT * FROM performance_schema.replication_applier_status_by_coordinator\G
    SELECT * FROM performance_schema.replication_applier_status_by_worker\G
    SELECT * FROM performance_schema.error_log;

    # check replication lag
    select * from sys.replication_lag;
    select * from sys.replication_status_full\G
    ```    
