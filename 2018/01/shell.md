> 数据库备份shell脚本

```shell
#!/bin/bash
#Author zhangxinzheng
MAXIMUM_BACKUP_FILES=10              #最大备份文件数
BACKUP_FOLDERNAME="database_backup"  #数据库备份文件的主目录
DB_HOSTNAME="localhost"              #mysql所在主机的主机名
DB_USERNAME="root"                   #mysql登录用户名
DB_PASSWORD=""                 #mysql登录密码
DATABASES=(
            "cms"
            "upms"                    #备份的数据库名
)
#=========
echo "Bash Database Backup Tool"
#CURRENT_DATE=$(date +%F)
CURRENT_DATE=$(date +%F)              #定义当前日期为变量
BACKUP_FOLDER="${BACKUP_FOLDERNAME}_${CURRENT_DATE}" #存放数据库备份文件的目录
mkdir $BACKUP_FOLDER #创建数据库备份文件目录
#统计需要被备份的数据库
count=0
while [ "x${DATABASES[count]}" != "x" ];do
    count=$(( count + 1 ))
done
echo "[+] ${count} databases will be backuped..."
#循环这个数据库名称列表然后逐个备份这些数据库
for DATABASE in ${DATABASES[@]};do
    echo "[+] Mysql-Dumping: ${DATABASE}"
    echo -n "   Began:  ";echo $(date)
    if $(mysqldump -h ${DB_HOSTNAME} -u${DB_USERNAME} -p${DB_PASSWORD} ${DATABASE} > "${BACKUP_FOLDER}/${DATABASE}.sql");then
        echo "  Dumped successfully!"
    else
        echo "  Failed dumping this database!"
    fi
        echo -n "   Finished: ";echo $(date)
done
echo
echo "[+] Packaging and compressing the backup folder..."
tar -cv ${BACKUP_FOLDER} | bzip2 > ${BACKUP_FOLDER}.tar.bz2 && rm -rf $BACKUP_FOLDER
BACKUP_FILES_MADE=$(ls -l ${BACKUP_FOLDERNAME}*.tar.bz2 | wc -l)
BACKUP_FILES_MADE=$(( $BACKUP_FILES_MADE - 0 )) 
#把已经完成的备份文件数的结果转换成整数数字
 
echo
echo "[+] There are ${BACKUP_FILES_MADE} backup files actually."
#判断如果已经完成的备份文件数比最大备份文件数要大，那么用已经备份的文件数减去最大备份文件数,打印要删除旧的备份文件
if [ $BACKUP_FILES_MADE -gt $MAXIMUM_BACKUP_FILES ];then
    REMOVE_FILES=$(( $BACKUP_FILES_MADE - $MAXIMUM_BACKUP_FILES ))
echo "[+] Remove ${REMOVE_FILES} old backup files."
#统计所有备份文件，把最新备份的文件存放在一个临时文件里，然后删除旧的文件，循环出临时文件的备份文件从临时目录里移到当前目录
    ALL_BACKUP_FILES=($(ls -t ${BACKUP_FOLDERNAME}*.tar.bz2))
    SAFE_BACKUP_FILES=("${ALL_BACKUP_FILES[@]:0:${MAXIMUM_BACKUP_FILES}}")
echo "[+] Safeting the newest backup files and removing old files..."
    FOLDER_SAFETY="_safety"
if [ ! -d $FOLDER_SAFETY ]
then mkdir $FOLDER_SAFETY
                                                                                                                    
fi
for FILE in ${SAFE_BACKUP_FILES[@]};do
                                                                                                                      
    mv -i  ${FILE}  ${FOLDER_SAFETY}
done
    rm -rf ${BACKUP_FOLDERNAME}*.tar.bz2
    mv  -i ${FOLDER_SAFETY}/* ./
    rm -rf ${FOLDER_SAFETY}
#以下显示备份的数据文件删除进度，一般脚本都是放在crontab里，所以我这里只是为了显示效果，可以不选择这个效果。
     
CHAR=''
for ((i=0;$i<=100;i+=2))
do  printf "Removing:[%-50s]%d%%\r" $CHAR $i
        sleep 0.1
CHAR=#$CHAR
done
    echo
fi
```

> jar包启动shell脚本

```shell
#! /bin/bash
echo stop application
PID=$(ps -ef | grep /data/application.jar | grep -v grep | awk '{ print $2 }')
if [ -z "$PID" ]
then 
        echo Application is already stopped
else 
        echo kill $PID
        kill -9 $PID
fi

echo start application

[ -d /data/log ] || mkdir -p /data/log;touch /data/log/spring.log
exec java -jar /data/application.jar >/data/log/spring.log &
```

> jpsall

```shell
#!/bin/bash
for host in hadoop102 hadoop103 hadoop104
do
 echo =============== $host ===============
 ssh $host jps 
done
```

> xsync
```shell
#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
 echo Not Enough Arguement!
 exit;
fi
#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
 echo ==================== $host ====================
 #3. 遍历所有目录，挨个发送
 for file in $@
 do
   #4. 判断文件是否存在
   if [ -e $file ]
     then
       #5. 获取父目录
       pdir=$(cd -P $(dirname $file); pwd)
       #6. 获取当前文件的名称
       fname=$(basename $file)
       ssh $host "mkdir -p $pdir"
       rsync -av $pdir/$fname $host:$pdir
     else
       echo $file does not exists!
   fi
 done
done
```

> myhadoop.sh
```shell
#!/bin/bash
if [ $# -lt 1 ]
then
 echo "No Args Input..."
 exit ;
fi
case $1 in
"start")
 echo " =================== 启动 hadoop 集群 ==================="
 echo " --------------- 启动 hdfs ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
 echo " --------------- 启动 yarn ---------------"
 ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
 echo " --------------- 启动 historyserver ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
 echo " =================== 关闭 hadoop 集群 ==================="
 echo " --------------- 关闭 historyserver ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
 echo " --------------- 关闭 yarn ---------------"
 ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
 echo " --------------- 关闭 hdfs ---------------"
 ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
 echo "Input Args Error..."
;;
esac
```
