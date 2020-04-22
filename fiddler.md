# fiddler 抓包教程
# 教程 https://blog.csdn.net/jianglianye21/article/details/81743129
## fiddler>Tools>Fiddler Options>Connections 勾选Allow remote computers to connect,设置端口为8888
## fiddler>Tools>Fiddler Options>https:选择 ... from remote clients only (只读客户端的)
## 连上电脑后，手机可能连不上网，解决方式 （1）打开注册表，在HKEY_CURRENT_USER\SOFTWARE\Microsoft\Fiddler2下创建一个DWORD，值设置为80
## 编写FiddlerScript rule，点击Rules > Customize Rules,用ctr+f查找到 OnBeforeRequest 方法添加一行代码.
```java
if (oSession.host.toLowerCase() == "webserver:8888") 
{
     oSession.host = "webserver:80";
}
```
## 添加代码后需要重启fiddler客户端

## 打开电脑手机热点或者连接局域网wifi
## 设置手动代理，ip为电脑的局域网ip,端口为步骤二设置的8888,
## 访问网站 电脑ip:8888 访问点击FiddlerRoot certificate下载证书，下载完成点击对应的app进行抓包