
## SQL Server最小配置启动修改内存
> 以前有段时间发现Sqlserver一直占用很大的内存所以把服务器的Sqlserver最大内存设置成4G内存
> 最近突然要回滚一个备份 结果发现内存低效率也不够 就手贱给设置了个0 结果变成默认最小内存了。

结果mssql服务直接启动失败 效果是这样的：
![](/images/0001.png?raw=true)
![](/images/0002.png?raw=true)
![](/images/0003.png?raw=true)
![](/images/0004.png?raw=true)

去谷歌找到解决方案
首先找到mssql的启动目录
然后 
> sqlservr.exe -f -s 你的实例名 

我的为:

> sqlservr.exe -f -s 192.168.1.218\MigHost

-f 参数是以最小配置启动SQL Server实例的意思

> sqlcmd -S SERVERNAME -U USERNAME -P PASSWORD

成功登入后输入sql语句

``` sql
sp_configure 'show advanced options', 1;
GO
RECONFIGURE;
GO
sp_configure 'max server memory', 最大内存数;
GO
RECONFIGURE;
GO
```
执行完毕后 从dos窗口关闭启动的sqlserver.exe 与sqlcmd。这时就可以直接在配置工具里启动Sqlserver服务了。