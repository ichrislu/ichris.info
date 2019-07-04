---
title: "基于容器的MySQL定时备份数据库"
date: "2018-04-12 11:37:00"
lastMod: "2018-04-12 11:37:00"
tags: ["docker", "备份", "mysql", "mysqldump"]
---

**注：以下使用mysqldump方式备份会导致锁表，生产环境备份建议使用xtrabackup**

1. 将如下内容添加至：/data/backup.sh

```shell
docker exec -i data_mysql_1 mysqldump -uroot -pPassword dbname > /data/`date +%Y%m%d`.sql \
	&& tar -zcvf `date +%Y%m%d`.tar.gz `date +%Y%m%d`.sql \
	&& rm -fr /data/`date +%Y%m%d`.sql
```

2. 添加执行权限：

```shell
sudo chmod +x /data/backup.sh
```

3. 在宿主机加定时调度，每天凌晨1点执行：crontab -e

```shell
0 1 * * * /bin/bash /data/backup.sh
```

docker exec -i不能加t，也不需要加t！！!这里坑了很久！