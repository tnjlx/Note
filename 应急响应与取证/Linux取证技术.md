- [1. Linux自启动](#1-linux自启动)
  - [1.1. init](#11-init)
  - [1.2. 配置文件](#12-配置文件)
    - [1.2.1. 系统](#121-系统)
    - [1.2.2. 用户](#122-用户)
  - [1.3. systemd](#13-systemd)
  - [1.4. 计划任务](#14-计划任务)
  - [1.5. 预加载库](#15-预加载库)
  - [1.6. 图形化相关](#16-图形化相关)
- [2. 进程信息](#2-进程信息)

# 1. Linux自启动
## 1.1. init
* /etc/rc.local
* /etc/init.d/rcS
* /etc/init.d/functions
* /etc/rc[0-6|S].d/*
* /etc/rc.d/rc.local
* /etc/rc.d/rc.sysinit
* /etc/rc.d/rc[0-6].d/*

## 1.2. 配置文件
### 1.2.1. 系统
* /etc/profile（用户登录时执行）
* /etc/profile.d（被/etc/profile调用）
* /etc/bashrc（用户登录时、打开bash shell时执行）
* /etc/bash.bashrc（用户登录时、打开bash shell时执行）

### 1.2.2. 用户
* ~/.profile（用户登录时执行）
* ~/.bash_login（用户登录时执行）
* ~/.bash_profile（用户登录时执行）
* ~/.bashrc（用户登录时、打开bash shell时执行）
* ~/.bash_logout（关闭bash shell时执行）

## 1.3. systemd
* 系统：/etc/systemd/system/*.service
* 用户：/usr/lib/systemd/system/*.service

## 1.4. 计划任务
* 系统：/etc/crontab
* 用户：/var/spool/cron/*

## 1.5. 预加载库
* /etc/ld.so.preload
* 环境变量：$LD_PRELOAD

## 1.6. 图形化相关
* gnome-session-properties机制，对应文件：~/.config/autostart/*
* $XDG_CONFIG_DIRS/autostart/*.desktop（当用户登陆桌面环境时启动）
* $XDG_CONFIG_HOME/autostart/*.desktop（当用户登陆桌面环境时启动），覆盖$XDG_CONFIG_DIRS/autostart目录下的同名文件

# 2. 进程信息
