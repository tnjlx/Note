- [1. 检查启动项](#1-检查启动项)
- [2. 检查驱动](#2-检查驱动)

# 1. 检查启动项
* 查看定时任务：crontab -l
* 查看ana定时任务：cat /etc/anacrontab
* 查看所有服务：service --status-all
* 检查最近天内修改的文件：find /usr/bin/ /usr/sbin/ /bin/ /usr/local/bin/ -type f -mtime +7 | xargs ls -la
* 检查是否存在病毒守护进程：lsof -p [pid]
* 检查是否存在病毒守护进程：strace -tt-T -etrace=all-p$pid
* 
# 2. 检查驱动
* 枚举/扫描系统驱动：lsmod
* wget ftp://ftp.pangeia.com.br/pub/seg/pac/chkrootkit.tar.gztar zxvf chkrootkit.tar.gzcd chkrootkit-0.52make sense./chkrootkit
* rkhunter
    * Wgethttps://nchc.dl.sourceforge.net/project/rkhunter/rkhunter/1.4.4/rkhunter-1.4.4.tar.gz
    * tar -zxvf rkhunter-1.4.4.tar.gz
    * cd rkhunter-1.4.4
    * ./installer.sh --install
    * rkhunter -c
