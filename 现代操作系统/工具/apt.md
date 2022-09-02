## apt 和apt-get的区别

对于ubuntu 16.04及以后的系统使用apt，之前的系统使用apt-get。这两个都是开源命令行工具，用于安装、更新和删除。**大多数的apt命令必须以sudo身份运行**。

## 常用指令

``` shell
#在使用apt命令之前都要确保包数据库本地是最新的
sudo apt update
# 使用以下命令可以查看即将升级的包
apt list --upgradeable

# 安装可更新的所有包
sudo apt upgrade # 更新所有可用的包
sudo apt full-upgrade # 将会删除已经安装的包，再全面升级整个系统

#安装指定包
sudo apt install <包> # 在更新数据库后，使用这个命令安装指定的包
sudo apt install <包1> <包2> # 同时安装多个

#删除
sudo apt remove <包> # 删除指定的包
sudo apt remove <包1> <包2> # 删除多个包，使用空格间隔
sudo apt purge <包> # remove卸载软件后会保留配置文件，purge不仅会删除包，而且删除所有的配置文件
sudo apt autoremove # 删除所有不需要的包

#在可用包中进行搜索
sudo apt search <包>

# 显示包的详细信息
apt show <包>
```



