#hadoop脚本分析

#start-all.sh
```sh
#取出hadoop的环境变量设置给HADOOP_BIN_PATH$(hadoop安装的位置/sbin)
#hadoopbin目录的提取
if not defined HADOOP_BIN_PATH ( 
  set HADOOP_BIN_PATH=%~dp0
)
```


