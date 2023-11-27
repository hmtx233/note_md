
#2023-11-20 #shell脚本

```shell
#!/bin/bash
time=`date +%F`
cd `dirname $0`

# 应用名（boot jar包名）

APP_NAME=app-1.0

JAR_NAME=$APP_NAME\.jar

PID=$APP_NAME\.pid
PORT=7002

APP_HOME=`pwd`

LOG_PATH=$APP_HOME/logs

JVM_OPTIONS="-server -Xms1g -Xmx1g -XX:MetaspaceSize=256m"

if [ ! -d $LOG_PATH ]; then
    mkdir $LOG_PATH
fi

#脚本参数提示，用来提示输入参数
usage() {
    echo "Usage: sh [run_script].sh [start|stop|restart|status]"
    exit 1
} 

#检查jar包是否存在于当前目录下
jar_exist() {
NOW_PATH=`pwd`
if [ ! -f "$JAR_NAME" ];then
	echo "您当前所在位置是$NOW_PATH"
	echo "在当前位置没到找$JAR_NAME,请检查jar包是否存在于当前位置(执行sh脚本的位置)！！！"
	echo "执行脚本的命令需要在jar包所在目录执行！！！"
	exit 404;
else
	echo “$APP_HOME/$JAR_NAME”
fi
}

#检查程序是否在运行
is_exist(){
  jar_exist
  pid=`ps -ef|grep $APP_HOME/$JAR_NAME|grep -v grep|awk '{print $2}' `
  #如果不存在返回1，存在返回0     
  if [ -z "${pid}" ]; then
   return 1
  else
    return 0
  fi
}

#启动方法
start(){
  jar_exist
  is_exist
  if [ $? -eq "0" ]; then 
    echo "--- ${JAR_NAME} is already running PID=${pid} ---" 
  else 
      echo "JVM_OPTIONS : "
      echo "$JVM_OPTIONS"
    nohup /usr/local/jdk1.8.0_191/bin/java  -jar  $JVM_OPTIONS  $APP_HOME/$JAR_NAME --spring.profiles.active=test --server.port=$PORT >>$LOG_PATH/${JAR_NAME}_${time}.log 2>&1 &
    echo $! > $PID
    echo "--- start $JAR_NAME successed PID=$! ---" 
   fi
  }

#输出运行状态
status(){
  is_exist
  if [ $? -eq "0" ]; then
    echo "--- ${JAR_NAME} is running PID is ${pid} ---"
  else
    echo "--- ${JAR_NAME} is not running ---"
  fi
}
#停止方法
stop(){
  jar_exist
  is_exist
  if [ $? -eq "0" ]; then 
    echo "--- app 2 PID = $pid begin kill -9 $pid  ---"
    kill -9  $pid
    sleep 2
    echo "--- $JAR_NAME process stopped ---"  
  else
    echo "--- ${JAR_NAME} is not running ---"
  fi  
}

#重启
restart(){
  stop
  start
}


#根据输入参数，选择执行对应函数，不输入则执行使用说明
case "$1" in
  "start")
    start
    ;;
  "stop")
    stop
    ;;
  "status")
    status
    ;;
  "restart")
    restart
    ;;
  *)
    usage
    ;;
esac
exit 0
t 0
;
esac
exit 0

```