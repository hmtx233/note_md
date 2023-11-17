# jdk配置

## 下载jdk

[Java Downloads | Oracle 中国](https://www.oracle.com/cn/java/technologies/downloads/#java17)
## 解压配置

```shell

# 创建目录
cd /opt/
mkdir jdk
cd /opt/jdk #压缩包 jdk-17_linux-x64_bin.tar.gz 放到这里

# 解压
tar -zxvf jdk-17_linux-x64_bin.tar.gz

# 打开配置文件
vim /etc/profile

# jdk path 写到 /etc/profile 最后面
export JAVA_HOME=/opt/jdk/jdk-17.0.9
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

# 生效配置文件
source /etc/profile
# 校验
java --version
```

# maven配置

## 下载maven

[Maven – Download Apache Maven](https://maven.apache.org/download.cgi)
## 解压配置

```shell

# 创建目录
cd /opt/
mkdir maven
cd /opt/maven #压缩包 apache-maven-3.9.5-bin.tar.gz 放到这里

# 解压
tar -zxvf apache-maven-3.9.5-bin.tar.gz

# 打开配置文件
vim /etc/profile

# maven path 写到 /etc/profile 最后面
export MAVEN_HOME=/opt/maven/apache-maven-3.9.5
export PATH=$MAVEN_HOME/bin:$PATH


# 生效配置文件
source /etc/profile

# 校验
mvn -v 
```