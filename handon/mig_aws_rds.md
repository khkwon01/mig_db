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

### 2. 

