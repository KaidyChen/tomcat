# tomcat
tomcat install and config 

1.upload the tomcat.tar.gz and unzip

2.setting firewall rules and open 8080 port
  $firewall-cmd --zone=public --add-port=8080/tcp --permanent

3.start the tomcat
  $cd /usr/local/tomcat/
  $./bin/startup.sh

4.test the tomcat service
  connect http://ip:8080

5.make config
5.1.需要配置：

Tomcat/conf/tomcat-users.xml加入：

<role rolename="manager"/>     
<role rolename="admin"/> 
<role rolename="admin-gui"/>
<role rolename="manager-gui"/>
<user username="xxx" password="***" roles="admin-gui,manager-gui"/>

以上配置好后本地可以访问，http://127.0.0.1:8080/manager.html



5.2.另外，需要修改Tomcat/webapps/manager/META-INF/context.xml文件：

<Context antiResourceLocking="false" privileged="true" >
<!--
  Remove the comment markers from around the Valve below to limit access to
  the manager application to clients connecting from localhost
-->


<Valve className="org.apache.catalina.valves.RemoteAddrValve"

 allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|\d+\.\d+\.\d+\.\d+" />

</Context>

或注释Value节点（tomcat9.0以下默认是注释的，所以不需修改）

<!--
<Valve className="org.apache.catalina.valves.RemoteAddrValve"

 allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
6.make Start up setting

方法一：
6.1.创建tomcat自动启动命令脚本

vi /etc/init.d/tomcat

6.2.写以下代码:

注意修改JAVA_HOME和CATALINA_HOME CATALINA_BASE字段, 匹配自己的安装路径 。

#!/bin/sh  
# chkconfig: 345 99 10  
# description: Auto-starts tomcat  
# /etc/init.d/tomcatd  
# Tomcat auto-start  
# Source function library.  
#. /etc/init.d/functions  
# source networking configuration.  
#. /etc/sysconfig/network  
RETVAL=0  
export JAVA_HOME=/usr/java/jdk1.8
export CATALINA_HOME=/usr/local/tomcat  
export CATALINA_BASE=/usr/local/tomcat  
start()  
{  
        if [ -f $CATALINA_HOME/bin/startup.sh ];  
          then  
            echo $"Starting Tomcat"  
                $CATALINA_HOME/bin/startup.sh  
            RETVAL=$?  
            echo " OK"  
            return $RETVAL  
        fi  
}  
stop()  
{  
        if [ -f $CATALINA_HOME/bin/shutdown.sh ];  
          then  
            echo $"Stopping Tomcat"  
                $CATALINA_HOME/bin/shutdown.sh  
            RETVAL=$?  
            sleep 1  
            ps -fwwu root | grep tomcat|grep -v grep | grep -v PID | awk '{print $2}'|xargs kill -9  
            echo " OK"  
            # [ $RETVAL -eq 0 ] && rm -f /var/lock/...  
            return $RETVAL  
        fi  
}  

case "$1" in  
 start)   
        start  
        ;;  
 stop)    
        stop  
        ;;  

 restart)  
         echo $"Restaring Tomcat"  
         $0 stop  
         sleep 1  
         $0 start  
         ;;  
 *)  
        echo $"Usage: $0 {start|stop|restart}"  
        exit 1  
        ;;  
esac  
exit $RETVAL  

6.3.设置执行权限 
chmod a+x /etc/init.d/tomcat

6.4.注册成服务 
chkconfig –add tomcat

6.5.设置开机启动 
chkconfig tomcaton

方法二：
7、设置jdk环境变量
vi /etc/profile
#在底部添加
JAVA_HOME=/usr/local/javajdk
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
export PATH JAVA_HOME CLASSPATH
source /etc/profile

8、启动tomcat
/usr/local/tomcat/bin/startup.sh
#/usr/local/tomcat/bin/shutdown.sh
可能要防火墙添加8080端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent
sudo firewall-cmd --reload

9、设置开机自启动

如果要开机自启动tomcat，配置如下：

chmod +x /etc/rc.d/rc.local

vi /etc/rc.d/rc.local

在文件中添加下面几行：

export JAVA_HOME=/usr/local/javajdk

export PATH=$JAVA_HOME/bin:$PATH

export CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar

/usr/local/tomcat/bin/startup.sh
