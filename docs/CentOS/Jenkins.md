### 1. 删除之前安装的jenkins(如果没安装跳过这一步)
```
# 查询是否安装过jenkins
rpm -qa |grep jenkins
# 卸载
rpm -e jenkins
# 彻底删除jenkins残留文件
find / -iname jenkins | xargs -n 1000 rm -rf
# 检查是否卸载成功
rpm -ql jenkins 
```
### 2.安装Java环境(根据自己选择的jenkins版本要选择对应Java版本)
###### ①卸载
```
#查看
rpm -qa | grep java
rpm -qa | grep jdk
#批量卸载
rpm -qa | grep jdk | xargs rpm -e --nodeps
rpm -qa | grep java | xargs rpm -e --nodeps
```
###### ②安装(我这里选择的是11版本)
```
1.安装
    yum search java-11-openjdk									# 可以不用执行，查找安装包
    yum install -y java-11-openjdk java-11-openjdk-devel		# 安装
2.查找JAVA安装目
    # 查找安装目录
    which java 或 ls -l $(which java)

    # 如果显示的是/usr/bin/java请执行下面命令
    ls -lr /usr/bin/java
    ls -lrt /etc/alternatives/java

    # 输出：/etc/alternatives/java ->  /usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.el7_9.x86_64/bin/java
    # 上面的/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.el7_9.x86_64就是JAVA的安装路径
3.配置环境变量    
    vi /etc/profile
    # 按" i "键进行编辑，设置环境变量，ESC退出编辑，" :wq "保存内容
    # Java Environment
    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.el7_9.x86_64
    export JRE_HOME=$JAVA_HOME/jre
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/jre/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
    export PATH=$JAVA_HOME/bin:$PATH
4 .使环境变量生效
    source /etc/profile


```

### 3. 安装jenkins (/usr/local/jenkins)
##### ①[war包](https://www.jenkins.io/download/)安装
```
1.把Jenkins.war放入到文件夹
    mkdir /usr/local/jenkins
2.创建start-jenkins文件
    # 创建文件
    vim start-jenkins.sh 文件内容 ➡ nohup java -jar jenkins.war --httpPort=8888 >output 2>&1 &
3.启动jenkins
    sh start-jenkins.sh
4.访问jenkins(请放开相应端口权限)
    http://IP地址:8888
5.默认密码查看
    vi /root/.jenkins/secrets/initialAdminPassword 
```
