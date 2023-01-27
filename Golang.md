### 搭建goframe框架

> 利用GoFrame CLI创建Go初始化项目

### 存储下载日志可利用框架提供日志处理

> 日志参数

```
logger:
  path:                  "/var/log/"   # 日志文件路径。默认为空，表示关闭，仅输出到终端
  file:                  "{Y-m-d}.log" # 日志文件格式。默认为"{Y-m-d}.log"
  prefix:                ""            # 日志内容输出前缀。默认为空
  level:                 "all"         # 日志输出级别
  ctxKeys:               []            # 自定义Context上下文变量名称，自动打印Context的变量到日志中。默认为空
  header:                true          # 是否打印日志的头信息。默认true
  stdout:                true          # 日志是否同时输出到终端。默认true
  rotateSize:            0             # 按照日志文件大小对文件进行滚动切分。默认为0，表示关闭滚动切分特性
  rotateExpire:          0             # 按照日志文件时间间隔对文件滚动切分。默认为0，表示关闭滚动切分特性
  rotateBackupLimit:     0             # 按照切分的文件数量清理切分文件，当滚动切分特性开启时有效。默认为0，表示不备份，切分则删除
  rotateBackupExpire:    0             # 按照切分的文件有效期清理切分文件，当滚动切分特性开启时有效。默认为0，表示不备份，切分则删除
  rotateBackupCompress:  0             # 滚动切分文件的压缩比（0-9）。默认为0，表示不压缩
  rotateCheckInterval:   "1h"          # 滚动切分的时间检测间隔，一般不需要设置。默认为1小时
  stdoutColorDisabled:   false         # 关闭终端的颜色打印。默认开启
  writerColorEnable:     false         # 日志文件是否带上颜色。默认false，表示不带颜色
```

#### 测试情况

> 在controller层代码中嵌入打印日志信息

```
func (c *cHello) Hello(ctx context.Context, req *v1.HelloReq) (res *v1.HelloRes, err error) {
	g.Log("test").Debug(ctx,"This is a test");
	g.RequestFromCtx(ctx).Response.Writeln("Hello World!")
	return
}
```

> 之后结果请求打印日志存储到日志配置的文件夹下

#### 建立与es的连接

> ```
> go get github.com/olivere/elastic
> 注：拉取时es的v6不用加版本号，详细版本可看github托管的官网
> ```

```
url := "http://192.168.17.129:9200/"
client, err := elastic.NewClient(elastic.SetURL(url), elastic.SetSniff(false), elastic.SetTraceLog(logger))
```

> 防止es的自动切换为内网地址或者docker地址，导致无法连接服务

##### 查询并获取结果

```
get1, err := client.Get().
		Index("fofa").
		Id("1").
		Do(ctx)
	if err != nil {
		// Handle error
		//panic(err)
	}
	if get1.Found {
		fmt.Printf("Got document %s in version %d from index %s, type %s\n", get1.Id, get1.Version, get1.Index, get1.Type)
		fmt.Println(get1.Source) // 字符串数据
		u := model.FoFa{}
		json.Unmarshal(*get1.Source, &u) // 将byte转为struct
		fmt.Println(u)
	}
```

#### 获取批量结果集并放入切片

```
fofaInfoList:=[]model.FoFa{}
	var res *elastic.SearchResult
	res,err = client.Search("fofa").Do(context.Background())
	num := res.Hits.TotalHits
	if num>0{
		for _,hit := range res.Hits.Hits{
			var p model.FoFa
			json.Unmarshal(*hit.Source,&p)
			fofaInfoList = append(fofaInfoList, p)
		}
		fmt.Println(fofaInfoList)
	}
```

#### 切片数据存入文件中

```
func down() {
	file, _ := os.Create("test.txt")
	for _,v := range fofaInfoList {
		c, _ :=json.Marshal(v)
		fmt.Println(c)
		file.WriteString(string(c))
		file.WriteString("\n")
	}
}
```

#### 通过接口下载文件

```
s := g.Server()
	s.BindHandler("/", func(r *ghttp.Request) {
		r.Response.ServeFileDownload("test.txt")
	})
	s.SetPort(8999)
	s.Run()
```

#### 切换选择数据库对象并建立模型

```
m := g.DB("user").Model("user")
```

#### 写入保存相关信息

```
g.Model("log").Data(g.Map{"info": "singIn"}).Insert()
```

> replace into 是insert加强版，当插入时存在有此行数据会将数据删除后插入。（根据主键或唯一索引判断）

```
Save()
```

会进行更新保存

##### 插入格式可结构体

```
type User struct {
    Uid  int    `orm:"uid"`
    Name string `orm:"name"`
    Site string `orm:"site"`
}
user := &User{
    Uid:  1,
    Name: "john",
    Site: "https://goframe.org",
}
g.Model("user").Data(user).Insert()
```

##### 条件查询

```go
g.Model("user").Where("uid=?", 1).One()
```

> 查找uid=1的一条数据,使用占位符方式进行参数替换,(还有另一种关键字，对主键的只能识别，where("uid=?",1) 等同于 wherePri(1))

```go
g.Model("user").Where("age>? AND name like ?", g.Slice{18, "%john%"}).All()
```

利用切片传递参数

```go
g.Model("user").Where(g.Map{"uid" : 1, "age>" : 18}).One()
```

利用map传递参数

##### 复杂条件查询

插入格式化条件字符串，在in中需要插入条件有多个字符串

查询的字段找score>100 并且状态（status）为successed,completed

```go
Wheref(`score > ? and status in (?)`, 100, g.Slice{"succeeded", "completed"})
```

查询结果包装在结构体或数组中

```go
type User struct {
    Id         int
    Passport   string
    Password   string
    NickName   string
    CreateTime *gtime.Time
}
```

```go
var user *User
g.Model("user").Where("id", 1).Scan(&user)
```

这种方式在查询时会进行内存分配

> var user User{} 会提前分配内存

In条件查询的相关关键字

```
whereIn -- And In
```

```go
WhereNotIn -- And Not In
```

```go
WhereOrIn -- Or In
```

```go
WhereOrNotIn -- Or Not In
```

##### 模糊查询

```
WhereLike/WhereNotLike/WhereOrLike/WhereOrNotLike
```

使用方法同In条件查询关键字

#### 请求参数获取方式

```
func main() {
	s := g.Server()
	s.BindHandler("/", func(r *ghttp.Request) {
		r.Response.Write(r.Get("name"))
	})
	s.SetPort(8199)
	s.Run()
}
```

请求URL： http://127.0.0.1:8199/?name=john&name=smith

得到参数smith

##### 数组参数

请求URL： http://127.0.0.1:8199/?array[]=john&array[]=smith

得到参数

```
["john","smith"]
```

##### Map参数

请求URL： http://127.0.0.1:8199/?map[id]=1&map[name]=john

得到数据

```
{"id":1,"name":"john"}
```

##### 请求参数包装到结构体中

```
type RegisterReq struct {
	Name  string
	Pass  string `p:"password1"`
	Pass2 string `p:"password2"`
}

type RegisterRes struct {
	Code  int         `json:"code"`
	Error string      `json:"error"`
	Data  interface{} `json:"data"`
}
func main() {
	s := g.Server()
	s.BindHandler("/register", func(r *ghttp.Request) {
		var req *RegisterReq
		if err := r.Parse(&req); err != nil {
			r.Response.WriteJsonExit(RegisterRes{
				Code:  1,
				Error: err.Error(),
			})
		}
		// ...
		r.Response.WriteJsonExit(RegisterRes{
			Data: req,
		})
	})
	s.SetPort(8199)
	s.Run()
}

```

URL请求：http://127.0.0.1:8199/register?name=john&password1=123&password2=456

请求结果

```
{"code":0,"error":"","data":{"Name":"john","Pass":"123","Pass2":"456"}}
```

#### 数据返回-Redirect

`ReditectTo`用以引导客户端跳转到指定的地址，地址可以是一个本地服务的相对路径，也可以是一个完整的`HTTP`地址。

#### 文件下载（通过流的方式进行）

```
s := g.Server()
	s.BindHandler("/", func(r *ghttp.Request) {
		r.Response.ServeFileDownload("test.txt")
	})
	s.SetPort(8999)
	s.Run()
```

> 指定目录文件

### goframe定时任务

time.Duration类型实际代表的是持续的纳秒数

select 会判断在通道中是否成功 <- channel

```
golang中加载定时任务
https://www.bilibili.com/read/cv12179303
```

传参是告诉 Timer 需要等待多长时间，Timer 是带有一个缓冲的 channal，在定时时间到达之前，没有数据写入 Timer.C，读取操作会阻塞当前协程，到达定时时间时，会向 channel 写入数据（当前时间），阻塞解除，被阻塞的协程得以恢复运行，达到延时或者定时执行的目的。