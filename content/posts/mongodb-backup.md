---
title: mongodb backup
date: 2015-12-22 13:30:24
tags: ['backup', 'mongodb', 'cronjob']
---

有台机器准备2月份下架
记一个 mongodb 备份小脚本 :)

```bash
#!/bin/bash
# vim: set et sw=2 ts=2 sts=2 ff=unix fenc=utf8:

MONGO_DATABASE="_name_"

MONGO_HOST="_ip_"
MONGO_PORT="_prot_"
TIMESTAMP=`date +%Y-%m-%dT%H:%M:%S`
MONGODUMP_PATH="/usr/bin/mongodump"
BACKUPS_DIR="/data/dumps/"
BACKUP_NAME="$MONGO_DATABASE-$TIMESTAMP"

while test $# -gt 0
do
  case "$1" in
    -m) echo "backup mongthly and clear week_dir"
      #rm $BACKUPS_DIR"week/*"
      find $BACKUPS_DIR"week" -type f -name '*.tgz' -delete
      tar -czPf $BACKUPS_DIR"month/"$BACKUP_NAME.tgz $BACKUPS_DIR$MONGO_DATABASE
      ;;
    -w) echo "backup weekly"
      echo "tar -czPf $BACKUPS_DIR"week/"$BACKUP_NAME.tgz $BACKUPS_DIR$MONGO_DATABASE"
      ;;
    -d) echo "just dump"
      $MONGODUMP_PATH -d $MONGO_DATABASE --out $BACKUPS_DIR
      ;;
    *) echo "do nothing"
      ;;
  esac
  shift
done
```


```cronjob
# crontab -e
10 3 * * * /bin/bash $HOME/bin/mongobackup.sh -d > /dev/null 2>&1
20 3 * * 1 /bin/bash $HOME/bin/mongobackup.sh -w > /dev/null 2>&1
30 3 1 * * /bin/bash $HOME/bin/mongobackup.sh -m > /dev/null 2>&1
```

每天早上3点按天／周／月分别保存
