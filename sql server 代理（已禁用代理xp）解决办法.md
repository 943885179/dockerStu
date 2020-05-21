1、说明：

在SQL Server Management Studio中连接到SQL Server实例后，会显示“SQL Server 代理”节点。如果当前该实例的Agent服务没有启动，“SQL Server 代理”后边就会显示“(已禁用代理XP)”。“已禁用代理”从字面上不难理解，后边的“XP”有点让人费解了，这个服务跟Windows XP系统还有关系吗？呵呵，玩笑。到搜索引擎上搜了一下，没有相关的说明。在SQL Server联机丛书里边找了找有了答案。在SQLServer配置选项表中有一项“Agent XPs”，该项是用来确定SQL Server Management Studio中是否显示SQL Server代理节点下的子节点。在代理服务未启动的时候，该项默认为禁用；服务启动后该项又会被启用。

当然，在服务未启动时，只需将配置项的值设置为１即可显示子节点。同样，在服务启动后，将配置项的值设置为０则会隐藏子节点。启用该项的方法是，以SQL Server 2008为例：

(1)新建查询，执行以下语句：

sp_configure 'show advanced options', 1;
GO
RECONFIGURE;
GO
sp_configure 'Agent XPs', 1;

GO

(2)新建查询，执行以下语句：

RECONFIGURE

GO

(3)重启SQL Server，OK