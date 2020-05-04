# [安装MinGW](https://sourceforge.net/projects/mingw-w64/)
# 添加环境变量path `d:\mingw-64\mingw64\bin`
# [安装OCI](https://www.oracle.com/database/technologies/instant-client/microsoft-windows-32-downloads.html) 
- Basic Package	instantclient-basic-nt-19.6.0.0.0dbru.zip
- SDK Package	instantclient-sdk-nt-19.6.0.0.0dbru.zip
# 解压OCI,Basic解压，然后将sdk解压到basic下，添加环境变量instantclient，
# `go get github.com/wendal/go-oci8` 下载驱动
- 进入gopath下src的github.com/wendal/go-oci8/windows，将下面的pkg-config.exe文件赋值到minw64的bin文件下
- 在minw64的bin文件下添加文件夹pkg_config，然后将gopath下src的github.com/wendal/go-oci8/windows下oci8.pc复制到minw64的bin文件下添加文件夹pkg_config，并且编辑
```
# Package Information for pkg-config
prefix=修改为instantclient目录，如C:/androidtools/orcale/instantclient_19_6 需要修改
exec_prefix=修改为instantclient1目录，如C:/androidtools/orcale/instantclient_19_6 需要修改
libdir=${exec_prefix}
includedir=${prefix}/sdk/include/

Name: OCI
Description: Oracle database engine
Version: 11.2
Libs: -L${libdir} -loci
Libs.private: 
Cflags: -I${includedir}
```
# 添加环境变量 PKG_CONFIG_PATH={xxxxx}\mingw64\lib\pkg-config
# `go get github.com/mattn/go-oci8` 下载驱动，然后开始使用
```golang
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/mattn/go-oci8"
    "log"
)
func Tss() {
    // 为log添加短文件名,方便查看行数
    log.SetFlags(log.Lshortfile | log.LstdFlags)

    log.Println("Oracle Driver example")

    //os.Setenv("NLS_LANG", "")

    // 用户名/密码@IP:端口/实例名  
    db, err := sql.Open("oci8", "whyyzs/whyyzs@127.0.0.1:1521/ORCL")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    rows, err := db.Query("select * from bs_goodsclass")
    if err != nil {
        log.Fatal(err)
    }
	defer rows.Close()
	cols,err:=rows.Columns()
	for i := 0; i < len(cols); i++ {
		fmt.Print(cols[i])
		fmt.Print("\t")
	}
	fmt.Scanln()
   /* for rows.Next() {
        var f1 float64
        var f2 string
        rows.Scan(&f1, &f2)
        log.Println(f1, f2) // 3.14 foo
    }*/
/*
    // 先删表,再建表
    db.Exec("drop table sdata")
    db.Exec("create table sdata(name varchar2(256))")

    db.Exec("insert into sdata values('中文')")
    db.Exec("insert into sdata values('1234567890ABCabc!@#$%^&*()_+')")

    rows, err = db.Query("select * from sdata")
    if err != nil {
        log.Fatal(err)
    }

    for rows.Next() {
        var name string
        rows.Scan(&name)
        log.Printf("Name = %s, len=%d", name, len(name))
    }
    rows.Close()*/
}

```