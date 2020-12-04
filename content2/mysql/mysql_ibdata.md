title: mysql ibdata 太大
slug: mysql_ibdata
date: 2015-12-07


ibdata 太大无法缩小


1. Do a mysqldump of all databases, procedures, triggers etc except the mysql and performance_schema databases
2. Drop all databases except the above 2 databases
``` drop schema xxxxx  ```
3. Stop mysql
```service mysqld stop```
4. Delete ibdata1 and ib_log files
5. Start mysql
```service mysqld start```
6. Restore from dump


