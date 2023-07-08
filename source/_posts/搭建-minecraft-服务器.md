---
title: 搭建 Minecraft 服务器
tags:
  - Linux
  - 我的世界
  - 服务器
id: '121'
categories:
  - - 教程
date: 2021-06-08 12:48:22
cover: https://blog.imwcr.cn/wp-content/uploads/2021/06/pexels-panumas-nikhomkhai-1148820-scaled.jpg
coverWidth: 1200
coverHeight: 600
---

## 准备服务器

突然想玩 Minecraft 联机，于是就捣鼓了一下。首先肯定要准备服务器，想起之前做过一期视频，然后打开阿里云的学生机页面。

结果非常 Amazing 啊

曾经的 https://promotion.aliyun.com/ntms/act/campus2018.html 这个页面转跳到了[开发者成长计划 (aliyun.com)](https://developer.aliyun.com/plan/grow-up)

[![](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_11-25-08.jpg)](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_11-25-08.jpg)

就是说原来的活动没有了，换了个新活动。

仔细看看，似乎价格还不错

[![](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_11-26-42.jpg)](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_11-26-42.jpg)


[![](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_11-26-33.jpg)](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_11-26-33.jpg)

等到要付款才发现根本没有优惠，于是想起来腾讯也有这么一个活动。

[![](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_11-29-45.jpg)](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_11-29-45.jpg)

还是腾讯香，二话不说马上买了一个

经过坚苦的找资料，终于把mc给装上了

现在才发现远没有想的那么难

## 开始安装

### 准备java环境

```
#安装 Java
yum install java-1.8.0-openjdk
#检查版本
java -version
```

### 下载游戏服务器端文件

```
#创建个mc运行目录
cd /
mkdir minecraft
cd minecraft
```

然后下载游戏服务器端文件

1.  [Download server for Minecraft Minecraft](https://www.minecraft.net/zh-hans/download/server)
2.  [Downloads for Minecraft Forge for Minecraft](https://files.minecraftforge.net/net/minecraftforge/forge/)
3.  [SpigotMC - High Performance Minecraft](https://www.spigotmc.org/)

```
#下载服务器端文件
wget https://launcher.mojang.com/v1/objects/1b557e7b033b583cd9f66746b7a9ab1ec1673ced/server.jar
```

如果你是使用非原版版本，下载完成后，还需要手动构建，命令如下

```
#安装forge
java -jar forge-1.16.5-36.1.25-installer.jar --installServer

#构建水桶服务器
(超级久)
java -Xmx1024M -jar BuildTools.jar --rev 1.16.5
```

### 防火墙放行

[![](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_11-41-55.jpg)](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_11-41-55.jpg)

在配置页面找到对应的页面放行 **25565** 端口（Minecraft联机默认端口）

如果安装了宝塔面板之类的程序，也需要放行端口。

### 启动服务器

执行下方指令即可，其中1024指的是启动内存，2000指的是最大内存。

```
java -Xms1024m -Xmx2000m -jar server.jar
```

第一次启动后要同意协议，操作如下：

1.  在shell中输入 **vim eula.txt** 编辑 **eula.txt** 文件
2.  按 **i** 键，进入编辑模式
3.  把 **eula=false** 改为 **eula=true**
4.  按 **Esc** 键，输入 **:wq** 保存退出（冒号是命令的一部分，记得输入哦）

然后再次执行启动命令

```
java -Xms1024m -Xmx2000m -jar server.jar
```

## 准备连接

复制服务器的公网ip，粘贴到Minecraft中即可连接

[![](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_12-00-45.jpg)](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_12-00-45.jpg)

[![](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_12-01-47.jpg)](https://blog.imwcr.cn/wp-content/uploads/2021/06/Snipaste_2021-06-08_12-01-47.jpg)

## 更多设置

打开 **server.properties** 文件进行修改即可，下面是几个常用的设置：

*   player-idle-timeout 玩家多久没动被踢出服务器 （0 表示不限制，单位秒）
*   pvp 玩家互相伤害
*   enable-command-block 启用命令方块
*   max-players 最大玩家数
*   server-port 连接端口，默认25565
*   online-mode 是否开启正版验证
*   motd 服务器页面表语

修改后要记得保存并重启服务才可生效

## 进一步优化

### 24h不间断开服

当你把终端关闭时，服务器也会随之停止，这时候就需要安装一个 screen 来解决这个问题

```
#安装screen
yum install -y screen
#创建一个新screen
screen -R Minecraft
#按Ctrl+A+D退出当前screen
```

### 运维脚本

```
#!/bin/bash

# Latest edit : 2021-07-13
# Coded by yzy
# Updated by LC
# Updated by Yuan_Tuo<yuantuo666@gmail.com>

USERNAME='root'
BIN_PATH='/minecraft'

MAX_MEM='2000M'
MIN_MEM='1024M'
LOG="$BIN_PATH/record_status.log"

dir=$(ls -l ${BIN_PATH}/mojang/ awk '/^d/ {print $NF}')

WAITING_TIME=5

ME=`whoami`

#-----------------------------------------

if [ $ME != $USERNAME ] ; then
    echo "Please run the $USERNAME user."
    exit
fi

usertip(){
echo "Support version:"
        for i in $dir
        do
            echo $i
        done
        echo "Usage: $0 [start/stop/restart/status/direct/cleanMinecraftItem] [version]"
}

if [ ! -f "${BIN_PATH}/mojang/$2/server.jar" ];then
    echo "Not exist $2."
usertip
    exit
fi

BOOT_PATH="mojang/$2"
NAME="Minecraft_$2"

direct(){
    cd "${BIN_PATH}/${BOOT_PATH}"
    java -Xmx$MAX_MEM -Xms$MIN_MEM -jar "${BIN_PATH}/${BOOT_PATH}/server.jar" nogui
}

start(){
    if pgrep -u "$USERNAME" -f "$NAME" > /dev/null ; then
        echo "$NAME is already running!"
        exit
    fi
    
    echo "Starting $NAME..."
    
    cd "${BIN_PATH}/${BOOT_PATH}"
    screen -AmdS "$NAME" java -Xmx$MAX_MEM -Xms$MIN_MEM -jar "${BIN_PATH}/${BOOT_PATH}/server.jar" nogui
    
    echo $(date +%Y/%m/%d-%H:%M)_Start $NAME >> "$LOG"
    exit
}

stop(){
    if pgrep -u "$USERNAME" -f "$NAME" > /dev/null ; then
        echo "$NAME has stopped."
    else
        echo "$NAME is not running!"
        exit
    fi
    
    screen -S "$NAME" -X eval 'stuff "stop"\015'
    
    echo $(date +%Y/%m/%d-%H:%M)_Stop $NAME >> "$LOG"
    exit
}

restart(){
    if pgrep -u "$USERNAME" -f "$NAME" > /dev/null ; then
        screen -S "$NAME" -X eval 'stuff "stop"\015'
    fi
    
    echo "Please wait $WAITING_TIME seconds for restarting."
    sleep $WAITING_TIME
    
    cd "$BIN_PATH"
    screen -AmdS "$NAME" java -Xmx$MAX_MEM -Xms$MIN_MEM -jar "$BIN_PATH/$BOOT_FILE" nogui
    echo "$NAME has restarted."
    
    echo $(date +%Y/%m/%d-%H:%M)_Restart $NAME >> "$LOG"
    exit
}

status(){
    if pgrep -u "$USERNAME" -f "$NAME" > /dev/null ; then
        echo "$NAME is running."
        exit
    else
        echo "$NAME is not running."
        exit
    fi
}

cleanMinecraftItem(){
    if pgrep -u "$USERNAME" -f "$NAME" > /dev/null ; then
        screen -S "$NAME" -X eval 'stuff "say 30s 后将会清空掉落物，请捡起重要物品！"\015'
        sleep 30
        screen -S "$NAME" -X eval 'stuff "kill @e[type=minecraft:item]"\015'
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
        restart
    ;;
    status)
        status
    ;;
    direct)
        direct
    ;;
    cleanMinecraftItem)
        cleanMinecraftItem
    ;;
    *)
        usertip
esac
```

\[2021-07-13 00:07\]更新了下脚本，搞成了版本分离的样子，整理工作由 lc 完成。准备研究 PHP 控制服务器启动。（挖坑）