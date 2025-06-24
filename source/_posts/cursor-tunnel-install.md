---
title: Cursor Tunnel 安装
tags:
  - Linux
  - 服务器
  - 网路
categories:
  - - 网络
  - - note
date: 2025-05-30 16:00:00
---

每次安装 Cursor Tunnel 都要下载，很麻烦，所以写个 Blog 记录一下下载方法，方便自己。

## Steps

```bash
# wget https://api2.cursor.sh/updates/download-latest?os=cli-alpine-arm64 -O cursor-tunnel.tar.gz
wget https://api2.cursor.sh/updates/download-latest?os=cli-alpine-x64 -O cursor-tunnel.tar.gz

tar -xzvf cursor-tunnel.tar.gz

# first, set up the account.
./cursor tunnel
# choosen Github for auth
# after running, Ctrl + C to stop

# then, run the following command to start the tunnel
nohup ./cursor tunnel > cursor-tunnel.log 2>&1 &

# check the log
tail -f cursor-tunnel.log

# if you want to stop the tunnel, run the following command
pkill -f "cursor tunnel"
```

## Note

必须选择 GitHub 进行认证，因为 cursor 不支持微软验证。

## Reference

- https://github.com/getcursor/cursor/issues/1191