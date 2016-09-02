
#linux文件合并
cat file1 file2 > newfile

#ssh 文件夹的上传
scp -r 要上传的文件  root@serverip:/opt

#文件解压
.zip
解压：unzip FileName.zip
压缩：zip FileName.zip DirName

．tar.gz 和 .tgz
解压：tar zxvf FileName.tar.gz
压缩：tar zcvf FileName.tar.gz DirName

### ssh 无密码登陆主机
－主机A：192.168.1.110

－主机B：192.168.1.111 

－需要配置主机A无密码登录主机A，主机B

－先确保所有主机的防火墙处于关闭状态。

－在主机A上执行如下
　- $cd ~/.ssh
　- $ssh-keygen -t rsa
　　- 然后一直按回车键，就会按照默认的选项将生成的密钥保存在.ssh/id_rsa文件中。
　- $cp id_rsa.pub authorized_keys 
　　- 这步完成后，正常情况下就可以无密码登录本机了，即ssh localhost，无需输入密码。
　- $scp authorized_keys hadoop@192.168.1.111:/home/hadoop/.ssh
　  - 把刚刚产生的authorized_keys文件拷一份到主机B上
　- $chmod 600 authorized_keys
　　- 进入主机B的.ssh目录，改变authorized_keys文件的许可权限
