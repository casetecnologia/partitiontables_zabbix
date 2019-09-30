# partitiontables_zabbix
```
This is a script for table partitioning for Zabbix database on MySQL.
Supports Zabbix 2.2, 3.0, 3.2, 3.4 and 4.0, MySQL version 5.6, 5.7 and 8.0 
``` 
This project is the code provided in the third chapter of the second edition of Zabbix Enterprise Distributed Monitoring System. If you need to use it, please indicate the source and follow the Apache 2.0 open source license.

# 1. configuration
Before running this script, you should adjust the database settings and the data retention policy
```
# MySQL connect information
ZABBIX_USER="zabbix"
ZABBIX_PWD="zabbix"
ZABBIX_DB="zabbix"
ZABBIX_PORT="3306"
ZABBIX_HOST="127.0.0.1"
MYSQL_BIN="mysql"

# For how many days you will keep history data (30 by default)
HISTORY_DAYS=30

# For how many months you will keep trend data (12 by default)
TREND_MONTHS=12
```
# 2. run it
Running the script  
```
shell# bash partitiontables_zabbix.sh 
table history      create partitions p20180716
table history_log  create partitions p20180716
table history_str  create partitions p20180716
table history_text create partitions p20180716
table history_uint create partitions p20180716
table history      create partitions p20180717
table history_log  create partitions p20180717
table history_str  create partitions p20180717
table history_text create partitions p20180717
table history_uint create partitions p20180717
......
table trends       create partitions p201807
table trends_uint  create partitions p201807
table trends       create partitions p201808
table trends_uint  create partitions p201808
```
# 3. check partitions
Verify that the script executed successfully
```
shell# mysql -uzabbix -pzabbix zabbix
MariaDB [zabbix]> show create table history\G;
*************************** 1. row ***************************
Table: history
Create Table: CREATE TABLE `history` (
  `itemid` bigint(20) unsigned NOT NULL,
  `clock` int(11) NOT NULL DEFAULT '0',
  `value` double(16,4) NOT NULL DEFAULT '0.0000',
  `ns` int(11) NOT NULL DEFAULT '0',
  KEY `history_1` (`itemid`,`clock`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
/*!50100 PARTITION BY RANGE ( clock)
(PARTITION p20180716 VALUES LESS THAN (1531756799) ENGINE = InnoDB,
PARTITION p20180717 VALUES LESS THAN (1531843199) ENGINE = InnoDB,
PARTITION p20180718 VALUES LESS THAN (1531929599) ENGINE = InnoDB,
PARTITION p20180719 VALUES LESS THAN (1532015999) ENGINE = InnoDB,
PARTITION p20180720 VALUES LESS THAN (1532102399) ENGINE = InnoDB,
PARTITION p20180721 VALUES LESS THAN (1532188799) ENGINE = InnoDB,
PARTITION p20180722 VALUES LESS THAN (1532275199) ENGINE = InnoDB,
PARTITION p20180723 VALUES LESS THAN (1532361599) ENGINE = InnoDB) */
1 row in set (0.00 sec)
MariaDB [zabbix]>
```
If you see PARTITION, the partitions have been created successfully. 
Everything is OK.

# 4. set crontab
Regular daily tasks are implemented using linux crontab, make sure crond has started successfully.
Otherwise new partitions will not be created automatically.
```
shell# crontab -e
1 0 * * * /usr/sbin/partitiontables_zabbix.sh
Shell# chmod 700 /usr/sbin/partitiontables_zabbix.sh
```
