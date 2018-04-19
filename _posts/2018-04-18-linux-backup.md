---
title: "Linux 数据库和文件备份"
layout: post
date: 2018-04-18
tags:
- Mysql
- Linux
categories: LINUX
blog: true
excerpt: 数据备份是很重要的，对数据库备份和文件备份的要求不同，脚本也有所区别
---

# 数据库和文件备份

### 1、数据库备份

数据库备份需全量备份，每几小时备份一次，并且需要定时的清理过久的文件，避免磁盘满。记得并赋予脚本执行权限

数据库备份脚本如下：

```shell
#!/bin/bash

date=`date "+%Y%m%d"`
time=`date "+%Y%m%d%H%M%S"`
echo "start backup $time"

tempDir=/data/tmp
remoteDir=/data/backup/
ip="10.10.10.10"

[ ! -d $tempDir ] && mkdir -p $tempDir

backupFile="$time.test_mgr.sql"

/usr/local/services/mysqldump-5.6/mysqldump -hHOST -PPORT -uroot -pPASSWORD DATABASE_NAME --default-character-set=utf8 > $tempDir/$backupFile

tar -zcvf "$backupFile.tar.gz" $backupFile

scp -P 36000 -r "$backupFile.tar.gz" root@$ip:$remoteDir

rm -f "$backupFile.tar.gz"
```

数据库备份清理脚本如下：

```shell
#!/bin/bash

date=`date "+%Y%m%d" --date="-5 day"`
echo "start clear $date"

backupDir=/data/backup

cd $backupDir

for filename in `ls`
do
  filedate=${filename:0:8}
  if [ -f $filename -a $filedate -le $date ]
  then
	rm -f $filename
	echo "remove: $filename"
  fi
done

```

crontab配置如下（可能配置在两个不同服务器上）：

```shell
#每小时备份一次
0 * * * * /data/backup/database_backup.sh gd>> /data/backup/logs/database_backup.log 2>&1 &

#每天清理一次
0 2 * * * /data/release/sh/beian_tss_sng_com_database_clear.sh gd >> /data/release/sh/logs/database_clear.log 2>&1 &
```



### 2、文件备份

文件备份为增量，一般不用考虑数据清除问题

文件备份脚本如下，如备份图片

```shell
#!/bin/bash

yesterday=`date -d "yesterday" +%Y%m%d`  
yesterday_time=`date -d "yesterday" +%Y%m%d%H%M%S`
echo "start backup $yesterday_time"

tempDir=/data/tmp
remoteDir=/data/backup/image
ip="10.10.10.10"

[ ! -d $tempDir ] && mkdir -p $tempDir

backupImageFile="$yesterday_time.image.tar.gz"

fileDir=/data/release/image/$yesterday/

if  [ ! -d $fileDir ] ;then
    echo "Directory($fileDir) no exists！"
	exit
fi

tar -zcvf $tempDir/$backupImageFile -C $fileDir .

cd $tempDir

scp -P 36000 "$backupImageFile" root@$ip:$remoteDir

rm -f $backupImageFile
```

crontab配置如下：

```
0 2 * * * /data/backup/image_backup.sh >> /data/release//logs/image_backup.log 2>&1 &
```





