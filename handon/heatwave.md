### 1. login oci 
- login portal (http://cloud.oracle.com)
  ![image](https://github.com/user-attachments/assets/1e779354-62d8-4315-bd3a-21df8721effa)

- After verification, you will be signed in to Oracle Cloud!
  ![image](https://github.com/user-attachments/assets/89bcb25a-6575-4a38-a23a-82edc2c98189)

### 2. Create Compartment (Menu: Identity & Security > Compartments)
- Fristly, you have to create the compartment using Heatwave

### 3. Create Virtual Cloud Network (Menu: Networing > Virtual Cloud Networks)
- You set-up vcn using `Start VCN Wizard` in menu
- After making, you input open ip-addres and port in public firewall or private firewall

### 4. Create Heatwave service (Menu: Databases > DB systems)
- In below screen, you can click `Create DB system`
  ![image](https://github.com/user-attachments/assets/bdd86aaa-33dd-4666-b987-6db758416b46)
- In below screen,
  - According to HA use, you can choice either of "Production" or "Development or test" mode
    (if you select Production mode, Heatwave use HA functionality. otherwise, standalone)
  - Choose compartment using Heatwave and input db name
  ![image](https://github.com/user-attachments/assets/762dc4b4-6be8-4b45-8013-f50103972916)
- setup database admin name and password for managing heatwave db
  ![image](https://github.com/user-attachments/assets/f38d75c5-c26d-4290-92a5-4db276f2830e)
- configure networking using heatwave (DB only use private network)
  ![image](https://github.com/user-attachments/assets/163b0d34-5f7f-43cf-ac9f-46cc3135de55)
- configure hardware (shape)
  - if you need to enable OLAP, lakehouse, Gen-AI functionality, you have to enable "Enable HeatWave cluster" (default is enable)
  - if you use lakehosue, Gen-AI, you set-up over 512GB for heatwave cluster node
  - Shape details : MySQL hardware configuration, HeatWave cluster configuration : Heatwave cluster hardware configuration
  ![image](https://github.com/user-attachments/assets/e0a41c1c-dd4a-43ec-b770-64e67d81bf2e)

