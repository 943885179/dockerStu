# docker pull mcr.microsoft.com/mssql/server 安装image镜像
# docker images 查看是否下载成功
# docker run --name dockermssql  -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=943885179@qq.COM" -p 1433:1433 -d mcr.microsoft.com/mssql/server 容器安装，注意要双引号（windows）
# docer start dockermssql 启动sql server (如果设置setting  resources advanced 下的memory太小会闪退，建议设置到2g以上)
# docker exec -it dockermssql /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "943885179@qq.COM"  进入数据库 -S 服务器 -U 用户名 -P 密码（密码要使用双引号连接）
# 添加操作语句:备注每次查询都要使用单独一行输入GO 表示开始查询，如
```sql
>1  select 1 from id;
>2  go
``` 