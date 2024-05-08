# mysql批量创建表




互联网业务快速发展，经常遇到业务在设计表时只是单表，后面随着业务快速发展，单表数据量达到千万级别，甚至上亿级别。此时mysql单表性能下降，解决办法之一是分表。

批量创建多张表结构一样的表，方法有很多，本文介绍两种，仅供参考：

<!-- more -->

## 一、存储过程创建多张表

### 	1、创建一百张表 createTable.sql

```mysql
#创建数据库要先指定字符集，否则使用数据库的默认字符集
CREATE DATABASE db_test DEFAULT CHARACTER SET UTF8;

USE db_test;

CALL create_table();

#-------------------以下是存储过程--------------

DELIMITER $$  
  
CREATE  
    PROCEDURE `db_test`.`create_table`()  
    BEGIN  
  
         DECLARE i INT;  
         DECLARE table_name VARCHAR(30);   
         DECLARE table_pre VARCHAR(30);   
         DECLARE sql_text VARCHAR(3000);   
         SET i=0;  
         SET table_name='';  
         SET table_pre='test_table_';  
         SET sql_text='';  
         WHILE i<100 DO   
            SET table_name=CONCAT(table_pre,i);   
              
            SET sql_text=CONCAT('CREATE TABLE ', table_name, ' (
                   `uid` BIGINT(11) unsigned NOT NULL COMMENT \'统一ID\',
                   `name` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT \'姓名\',
                   `idcard` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT \'身份证\',
                   `idcard_url` varchar(256) COLLATE utf8_bin DEFAULT NULL COMMENT \'身份证图片\',
                   `idcard_auth_status` int(4) unsigned DEFAULT 0 COMMENT \'身份证认证状态\',
                   `idcard_auth_time` BIGINT(12) unsigned DEFAULT 0 COMMENT \'身份证认证时间\',
                   `phone` varchar(32) COLLATE utf8_bin DEFAULT NULL COMMENT \'手机号码\',
                   `phone_auth_status` int(4) unsigned DEFAULT 0 COMMENT \'手机认证状态\',
                   `phone_auth_time` BIGINT(12) unsigned DEFAULT 0 COMMENT \'手机认证时间\',
                   `update_time` BIGINT(12) unsigned DEFAULT 0 COMMENT \'更新时间\',
                   PRIMARY KEY (`uid`),
                   KEY `index_name` (`phone`(11)),
                   KEY `index_idcard` (`idcard`(18)),
                 ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin CHECKSUM=1 DELAY_KEY_WRITE=1 ROW_FORMAT=DYNAMIC' );  
                  
            SELECT sql_text;   
            SET @sql_text=sql_text;  
            PREPARE stmt FROM @sql_text;  
            EXECUTE stmt;  
            DEALLOCATE PREPARE stmt;    
            SET i=i+1;  
        END WHILE;  
  
    END$$  
            
DELIMITER ;

```

### 2、执行存储过程，创建表

```shell
登上mysql，执行createTable.sql即可
```



## 二、**利用sql语句创建多张表** 

```shell
#!/bin/sh

# 数据表名定义
timestamp=`date -d "3 month ago" +%Y%m%d`
tablename='t_table_test_20191108'

sql="CREATE TABLE $tablename (
        id BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '唯一id',
        uid BIGINT(20) UNSIGNED NOT NULL DEFAULT '0' COMMENT 'uid',
        msg_id VARCHAR(64) NOT NULL DEFAULT '' COMMENT '消息id',
        msg_ts BIGINT(20) NOT NULL DEFAULT '0' COMMENT '发言时间戳',
        sender_uid BIGINT(20) UNSIGNED NOT NULL DEFAULT '0' COMMENT '发言者uid',
        sender_nic VARCHAR(64) NOT NULL DEFAULT '' COMMENT '发言者昵称',
        text VARCHAR(2048) NOT NULL DEFAULT '' COMMENT '发言内容',
        update_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
        PRIMARY KEY (id),
        UNIQUE INDEX idx_msg_id (msg_id),
        INDEX idx_anchor_uid_msg_ts (anchor_uid, msg_ts)
)COLLATE='utf8mb4_general_ci' ENGINE=InnoDB AUTO_INCREMENT=1000;"



mysql -h 10.23.22.22 -udb_test -pdb_test -P2223 -Ddb_test --default-character-set=utf8 -Ne "$sql"
```

如果字段名和mysql关键字重复，将字段名用**`**括起来，例如：`index`

```shell
#!/bin/sh

# 数据表名定义
timestamp=`date -d "3 month ago" +%Y%m%d`
tablename='t_table_test_20191108'

sql="CREATE TABLE $tablename (
        \`id \` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '唯一id',
        \`uid \` BIGINT(20) UNSIGNED NOT NULL DEFAULT '0' COMMENT 'uid',
        \`msg_id\` VARCHAR(64) NOT NULL DEFAULT '' COMMENT '消息id',
        \`msg_ts\` BIGINT(20) NOT NULL DEFAULT '0' COMMENT '发言时间戳',
        \`sender_uid\` BIGINT(20) UNSIGNED NOT NULL DEFAULT '0' COMMENT '发言者uid',
        \`sender_nic\` VARCHAR(64) NOT NULL DEFAULT '' COMMENT '发言者昵称',
        \`text\` VARCHAR(2048) NOT NULL DEFAULT '' COMMENT '发言内容',
        \`update_time\` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
        PRIMARY KEY (\`id\`),
        UNIQUE INDEX idx_msg_id (\`msg_id\`),
        INDEX idx_anchor_uid_msg_ts (\`anchor_uid\`, \`msg_ts\`)
)COLLATE='utf8mb4_general_ci' ENGINE=InnoDB AUTO_INCREMENT=1000;"



mysql -h 10.23.22.22 -udb_test -pdb_test -P2223 -Ddb_test --default-character-set=utf8 -Ne "$sql"
```


