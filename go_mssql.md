# 拉取mssql `go get github.com/denisenkom/go-mssqldb`
# 案例
```golang
package util

import (
	"database/sql"
	"fmt"
	"log"
	"time"
	"encoding/json"

	_ "github.com/denisenkom/go-mssqldb"
)

var connString string

func Config(server string, port int, user string, password string, database string) {
	connString = fmt.Sprintf("server=%s;port=%d;database=%s;user id=%s;password=%s", server, port, database, user, password)
}
/*
* 只能查询一条数据的一个值
* select a,b from xxx 报错，只能有一列
* select a from xxx 指挥返回第一条数据的值
*/
func QueryScan(sqlStr string,data interface{}) (interface{}) {
	db, err := sql.Open("mssql", connString)
	if err != nil {
		fmt.Println("数据库连接失败:", err.Error())
		log.Fatal("Open Connection failed:", err.Error())
	}
	defer db.Close()
	if err != nil {
		fmt.Println("查询失败：", err.Error())
		log.Fatal("QUERYROW failed:", err.Error())
	}
	err = db.QueryRow(sqlStr).Scan(&data)
	if err != nil {
		log.Fatal("读取失败:", err.Error())
	}
	return data
	
}
/*
*修改语句
*/
func Exec(sqlStr string) bool  {
	db,err :=sql.Open("mssql",connString)
	if err != nil {
		fmt.Println(err)
	}
	defer db.Close()
	update, err :=db.Exec(sqlStr)
	if err != nil {
		fmt.Println(err)
	}
	if update==nil {
		return false
	}
	return true
}
/*
*
*/
func QueryJsonStr(sqlStr string)(string,error){
	db, err := sql.Open("mssql", connString)
	if err != nil {
		fmt.Println("数据库连接失败:", err.Error())
		log.Fatal("Open Connection failed:", err.Error())
		return "",err
	}
	defer db.Close()
	rows, err := db.Query(sqlStr)
	if err != nil {
		log.Fatal("Query failed:", err.Error())
		return "",err
	}
	defer rows.Close()
	columns,err :=rows.Columns()
	if err != nil {
		return "", err
	}
	count :=len(columns)
	tableData := make([]map[string]interface{},0)
	values := make([]interface{},count)
	valuePtrs := make([]interface{},count)
	//遍历每一行
	for rows.Next() {
		for i := 0; i < count; i++ {
		valuePtrs[i] = &values[i]
		}
		rows.Scan(valuePtrs...)
		entry := make(map[string]interface{})
		for i,col := range columns {
			var v interface{}
			val := values[i]
			b,ok := val.([]byte)
			if ok {
				v = string(b)
			} else {
				v = val
			}
			entry[col] = v
		}
		tableData = append(tableData,entry)
	}
	jsonData,err := json.Marshal(tableData)
	if err != nil {
		return "",err
	}
	fmt.Println(string(jsonData))
	return string(jsonData),nil 
}


//打印一行记录，传入一个行的所有列信息
func printRow(colsdata []interface{}) {
	for _, val := range colsdata {
		switch v := (*(val.(*interface{}))).(type) {
		case nil:
			fmt.Print("NULL")
		case bool:
			if v {
				fmt.Print("True")
			} else {
				fmt.Print("False")
			}
		case []byte:
			fmt.Print(string(v))
		case time.Time:
			fmt.Print(v.Format("2016-01-02 15:05:05.999"))
		default:
			fmt.Print(v)
		}
		fmt.Print("\t")
	}
	fmt.Println()
}

func test() {
	var isdebug = true
	var server = "localhost"
	var port = 1433
	var user = "sa"
	var password = "943885179@qq.COM"
	var database = "Test"
	//连接字符串
	connString := fmt.Sprintf("server=%s;port=%d;database=%s;user id=%s;password=%s", server, port, database, user, password)
	if isdebug {
		fmt.Println(connString)
	}
	//建立连接
	conn, err := sql.Open("mssql", connString)
	if err != nil {
		log.Fatal("Open Connection failed:", err.Error())
	}
	defer conn.Close()

	//产生查询语句的Statement
	stmt, err := conn.Prepare(`select * from userinfo`)
	if err != nil {
		log.Fatal("Prepare failed:", err.Error())
	}
	defer stmt.Close()

	//通过Statement执行查询
	rows, err := stmt.Query()
	if err != nil {
		log.Fatal("Query failed:", err.Error())
	}

	//建立一个列数组
	cols, err := rows.Columns()
	var colsdata = make([]interface{}, len(cols))
	for i := 0; i < len(cols); i++ {
		colsdata[i] = new(interface{})
		fmt.Print(cols[i])
		fmt.Print("\t")
	}
	//遍历每一行
	for rows.Next() {
		rows.Scan(colsdata...) //将查到的数据写入到这行中
		printRow(colsdata)     //打印此行
	}
	defer rows.Close()
}

```