![image](https://github.com/user-attachments/assets/7568374b-9deb-4f56-a28c-e8fb3351c804)

- source debezium tool 계정 생성
  ```
  mysql> CREATE USER 'user'@'%' IDENTIFIED BY 'password';
  mysql> GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'user'@'%';
  -- 만약 reload 권한이 안될 경우
  mysql> GRANT SELECT, flush_tables, lock tables, show dataabases, replication slave, replication client on *.* to 'user' IDENTIFIED BY 'password';
  or
  mysql> CREATE USER user@'%' IDENTIFIED BY '********' default role 'administrator';
  ```
