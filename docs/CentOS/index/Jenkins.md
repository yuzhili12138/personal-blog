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
##### [①war包安装](https://www.jenkins.io/download/)
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
##### ②yum安装
```
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io-2023.key
1.安装jenkins
    yum install jenkins
2.修改jenkins配置
    vi /etc/sysconfig/jenkins
        JENKINS_USER="root"  #将用户改成 root
        JENKINS_PORT="8888"  #修改默认端口
    (如果上面修改找不到文件，使用下面的这行修改端口)
    vi /usr/lib/systemd/system/jenkins.service
        JENKINS_PORT="8888"
3.修改完，记得刷新配置
    systemctl daemon-reload
4.添加自启动   
    systemctl enable jenkins
5.启动 Jenkins   (stop,restart)
    systemctl start jenkins
6.默认密码查看
    cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 4. 中文设置
```
1.安装Locale插件
2.System 里面 设置 默认语言先设置为 en_US
3.然后url加/restart 重启jenkins
4.System 里面 设置 默认语言改为zh_CN
```
