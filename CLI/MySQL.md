#MySQL
### Dump db
``` bash
mysqldump --opt --host=0.0.0.0 --user=username -ppassword db_name > backupDB/XXX.sql
```

### Start id from 1
``` bash
ALTER TABLE tbl AUTO_INCREMENT = 1;
```
### Create user and db
``` bash
CREATE USER "admin"@"localhost" IDENTIFIED BY "admin pass";
# "admin"@"%" for wildcard
CREATE DATABASE my_db CHARACTER SET utf8;
GRANT ALL ON my_db.* to admin@localhost;
GRANT SELECT, SHOW VIEW ON db.* TO 'db-readonly'@'%' IDENTIFIED BY 'Aa@123456' REQUIRE SSL;
FLUSH PRIVILEGES;
```
### Change pass user
``` bash
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
```
### Disable foreign key checks
``` bash
SET foreign_key_checks = 0;
```