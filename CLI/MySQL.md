#MySQL
### Dump db
``` sql
mysqldump --opt --host=0.0.0.0 --user=username -ppassword db_name > backupDB/XXX.sql
```

### Restore db from file
```bash
mysql - u Username -p dbNameYouWant < databasename_backup.sql;
```

### Start id from 1
``` sql
ALTER TABLE tbl AUTO_INCREMENT = 1;
```
### Users and Privileges
``` sql
CREATE USER "admin"@"localhost" IDENTIFIED BY "admin pass";
# "admin"@"%" for any host
CREATE DATABASE my_db CHARACTER SET utf8;
GRANT ALL ON my_db.* to admin@localhost;
GRANT SELECT, SHOW VIEW ON db.* TO 'db-readonly'@'%' IDENTIFIED BY 'Aa@123456' REQUIRE SSL;
# revoke permision
REVOKE ALL PRIVILEGES ON base.* FROM 'user'@'host'; -- one permission only
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'user'@'host'; -- all permissions
FLUSH PRIVILEGES;
# set password
SET PASSWORD = PASSWORD('new_pass');
SET PASSWORD FOR 'user'@'host' = PASSWORD('new_pass');
SET PASSWORD = OLD_PASSWORD('new_pass');
```
### Change pass user
``` sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
```
### Disable foreign key checks
``` sql
SET foreign_key_checks = 0;
```