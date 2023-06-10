# shell函数

## 函数语法

```bash
语法一:
函数名 () {
    代码块
    return N
    }
语法二：
function 函数名 {
      代码块
      return N
      }

```

-   直接使用函数名就可以调用函数

## 案例

```bash
#!/bin/bash
#ngingx service manager scripts
#=====================variables
nginxdoc="/usr/local/nginx"
nginxd="$nginxdoc/sbin/nginx"
pid="$nginxdoc/logs/nginx.pid"
conf="$nginxdoc/conf/nginx.conf"
#====================function
mystart () {
   if [ -f $pid ] && pstree -p |grep `cat $pid` &>/dev/null;then
      echo "nginx already run..."
      exit 0
   else
      if $nginxd ;then
         echo -e "nginx start\t\t\t\t[\033[32m OK \033[0m]"
      else
         echo -e "nginx start\t\t\t\t[\033[31m FAIL \033[0m]"
      fi
   fi
}
mystop () {
   if [ -f $pid ] && pstree -p |grep `cat $pid` &>/dev/null;then
      if killall -s QUIT $nginxd;then
            echo -e "nginx stop\t\t\t\t[\033[32m OK \033[0m]"
      else
         echo -e "nginx stop\t\t\t\t[\033[31m FAIL \033[0m]"
      fi
   else
    echo "nginx already stop...."
   fi
}
myrestart () {
  mystop;sleep 2 ;mystart
}
myreload () {
if [ -f $pid ] && pstree -p |grep `cat $pid` &>/dev/null;then
   if killall -s HUP $nginxd;then
      echo -e "nginx reload\t\t\t\t[\033[32m OK \033[0m]"
      else
         echo -e "nginx reload\t\t\t\t[\033[31m FAIL \033[0m]"
      fi
else
    echo "nginx is stop...."
fi
}
mystatus () {
if [ -f $pid ] && pstree -p |grep `cat $pid` &>/dev/null;then
   echo "nginx is open"
else
  echo "nginx is stop"
fi
}
#===================main
#calld function
case $1 in 
start|START)
    mystart
;;
stop) mystop ;;
restart) myrestart ;;
reload) myreload ;;
status) mystatus ;;
esac
```
