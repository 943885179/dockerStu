# 基础操作
# 进阶操作
>打开`show>show postman console` 查看请求详细信息

>Pre-request Script 选项可以输入一些js,例如变量赋值，输出等,url输入框:`{{basicUrl}}/xxxx`,为basicUrl赋值`pr.variables.set('basicUrl','http://localhost:5000/api')`
>变量优先级从低到高 global>conllection>environment>data>local
>pr.globals>ui赋值>pr.environment>data file>pr.variables
