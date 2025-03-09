---
title: 后端项目学习
tags:
  - simplebank
  - docker
  - sqlc
  - gin
  - database
  - go-mock
  - go-test
  - go-migrate
  - dockerfile
  - authority
  - restful-api
  - hash加密
  - 数据库事务
  - viper
  - proto
  - gRPC
---
## 工具：
1. https://doc.oschina.net/grpc?t=58008 （gRPC）
2. https://dbdiagram.io 可视化数据库工具
3. 
  
# 重新开启simplebank学习！！！



### 一.创建数据库表

https://dbdiagram.io 可视化数据库工具

#### 1.创建账户表

~~~go
Table accounts as A { //A作为account的别名
  id bigserisal [pk]  //pk作为主键 自增的id列
  owner varchar
  balance bigint
  currency varchar 
  created_at timestamp [default: `now()`] //自动获取时间
}
~~~

#### 2.创建条目表

//记录账户余额的变化

~~~go
Table entries {
  id bigint [pk] //
  account_id bigint [ref : > A.id] //外键 账户和条目之间是1对多关系
    amount bigint [not null note:`可以是负或者正`] //正负取决于取出还是存入 note是添加注释
  created_at timestamp [default: `now()`] //记录条目的创建时间
}
~~~

#### 3.创建 转账表

~~~go
Table transfers {
  id bigint [pk]
  from_account_id bigint [ref : > A.id]
  to_account_id bigint [ref : > A.id]
  amount  bigint [not null note: `一定不能为空`]//note为注释
  created_at timestamp [default: `now()`]
}
~~~

在此之后向列中添加非空约束 例如 ：

balance bigint [not null] //  **非空约束是一种用于限制数据库表中某列不能为空的约束**



枚举

~~~
Enum Currency{
USD
EUR
}
~~~



向表中添加索引

~~~go
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs

Table accounts as A {
  id bigserisal [pk]
  owner varchar [not null]
  balance bigint [not null]
  currency varchar  [not null]
  created_at timestamp [default: `now()`] 

  Indexes {
    owner
  }
}

Table entries {
  id bigint [pk]
  account_id bigint [ref : > A.id] 
  
  amount bigint [not null]
  created_at timestamp [default: `now()`]
  
 //列出特定账户的所有条目
Indexes {
  account_id
}
                        
}

Table transfers {
  id bigint [pk]
  from_account_id bigint [ref : > A.id]
  to_account_id bigint [ref : > A.id]
  amount  bigint [not null]
  created_at timestamp [default: `now()`]

  
Indexes {
  from_account_id
  to_account_id
  (from_account_id,to_account_id)
}
}

~~~

这些做好之后使用导出功能 生成代码

~~~go
CREATE TABLE "accounts" (
  "id" bigserisal PRIMARY KEY,
  "owner" varchar NOT NULL,
  "balance" bigint NOT NULL,
  "currency" varchar NOT NULL,
  "created_at" timestamp DEFAULT (now())
);

CREATE TABLE "entries" (
  "id" bigint PRIMARY KEY,
  "account_id" bigint,
  "amount" bigint NOT NULL,
  "created_at" timestamp DEFAULT (now())
);

CREATE TABLE "transfers" (
  "id" bigint PRIMARY KEY,
  "from_account_id" bigint,
  "to_account_id" bigint,
  "amount" bigint NOT NULL,
  "created_at" timestamp DEFAULT (now())
);

CREATE INDEX ON "accounts" ("owner");

CREATE INDEX ON "entries" ("account_id");

CREATE INDEX ON "transfers" ("from_account_id");

CREATE INDEX ON "transfers" ("to_account_id");

CREATE INDEX ON "transfers" ("from_account_id", "to_account_id");

//将外键添加到表中

ALTER TABLE "entries" ADD FOREIGN KEY ("account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("from_account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("to_account_id") REFERENCES "accounts" ("id");


~~~



### 二.Docker

**用指令创建容器的时候 一定要注意-p参数 将容器的端口映射到主机上 一定要保证端口不要被占用 否则将会产生问题**



拉取镜像语法

~~~shell
docker pull <image>:<tag>
~~~

开始一个容器指令

~~~shell
1.docker run --name <container_name> -e <environment_variable> -d <image>:tag
:
2.docker run --name some-postgres -e POSTGRES_PASSWORD=mysecret -d postgres 
#！！！端口映射 -p 5432:5432 //注意防止端口冲突自行更改
示例：
docker run --name postgres12 -e POSTGRES_USER=root -e POSTGRES_PASSWORD=secret -d postgres:12-alpine 

3. docker exec -it <contain_name_or_id> <comman> [args]
示例：#进入psql控制台
docker exec -it postgres12 psql -U root

4.显示容器日志
docker logs <container_name_or_id>
示例:
docker logs postgres12

5.连接shell
指令：
docker exec -it postgres12 /bin/sh
#创建新的数据库
createdb --username=root --owner=root simple_bank
#使用psql连接
 psql simple_bank
 #删除数据库
 dropdb [名称]
 
 exit退出shell
 
 #指令结合
 docker exec -it postgres12 createdb --username=root --owner=root simple_bank
 docker exec -it postgres12 psql -U root simple_bank
 
 #查找指令
 history | grep "docker run" //linux
 history | Select-String "docker run"//windows
 
~~~

区分 docker中的 镜像和容器

docker image中包含多个运行 容器的应用实例 类似结构：

\- docker image

- ├── container1

* ├──container2

* ├──container3

### 三.Tableplus

将sql文件导入到tableplus中

在tableplus中删除表 使用sql指令

~~~sql
DROP TABLE accounts CASCADE; //注意替换表名称
~~~





### 四.DB migration

迁移指令：

~~~
 migrate create -ext sql -dir db/migration -seq init_schema
~~~



up/down migration：理解迁移 类比栈结构 向上新数据表 向下 旧数据表

**使用migrate up指令时         Old DB 在文件中 一次按照 1.up.sql 2.up.sql 3.up.sql 依次运行到New DB**

**使用migrate down指令时  New DB 在文件中依次按照 3.up.sql 2.up.sql  1.up.sql 依次运行到Old DB**



**old DB schema** —–> migrate up —––> x.up.sql —–>**New DB schema**

​      <—————————-  x.down.sql <———migrate down<————-



将最开始的.sql文件放入 .up.sql中

### 五.Makefile文件

**创建规则后使用  make指令 快速创建**

如果你是萌新开始给到你一个项目 你可以通过makefile文件快速构建

~~~shell
migrate -help
#通过看日志 知道使用什么指令来工作
#迁移指令
migrate -path simplebank/db/migration -database "postgresql://root:secret@localhost:5432/simple_bank" -verbose up
#出现ssl错误
添加sslmode=disabled

#出现了一系列的迁移错误   解决方案
强制更改版本
migrate -path simplebank/db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" -verbose force 1
~~~



### 六.数据库的CRUD

DATAVASE/SQL库

GORM

sqlx（兼容多）

sqlc（最好的 融合了以上两者的优点）



### 七.使用sqlc

~~~
sqlc init
~~~

介绍:

sqlc 从 SQL 生成**类型安全的代码**。以下是它的工作原理：

1. 您使用 SQL 编写查询。
2. 运行 sqlc 来生成具有这些查询的类型安全接口的代码。
3. 编写调用生成的代码的应用程序代码。

查看[一个交互式示例](https://play.sqlc.dev/)来了解它的实际应用，以及 sqlc 背后的动机的[介绍性博客文章](https://conroy.org/introducing-sqlc)。



### 八.sqlc.yaml

~~~go
version: "2"
sql:
- schema: "simplebank/db/migration" //数据库表
  queries: "db/query" //数据库查询 首先要编写数据库查询
  engine: "postgresql" //使用的数据库
  gen:
    go: 
      package: "db"
      out: "simplebank/db/sqlc"
      sql_package: "pgx/v5"
      emit_json_tags: true
      emit_interface: false
      emit_empty_slices: true
      overrides:
        - db_type: "timestamptz"
          go_type: "time.Time"
        - db_type: "uuid"
          go_type: "github.com/google/uuid.UUID"
~~~

account.sql

~~~sql
-- name: CreateAccount :one
INSERT INTO accounts (
  owner,
  balance,
  currency
) VALUES (
  $1, $2, $3
) RETURNING *;
~~~



#### make sqlc 生成代码

##### 1.account.sql.go

##### 2.db.go

##### 3.models.go

**在生成之后由于没有 初始化项目 使得项目报红**

~~~shell
go mod init project/simplebank
go mod tidy
~~~





### 九.编写单元测试用例

1.导入未使用的包在前面添加_可以防止系统自动将它删除

例如：

_ "github.com/lib/pq "

错误

~~~
cannot use conn (variable of type *sql.DB) as DBTX value in argument to New: *sql.DB does not implement DBTX (wrong type for method Exec)
		have Exec(string, ...any) (sql.Result, error)
		want Exec(context.Context, string, ...interface{}) (pgconn.CommandTag,
		
        你遇到的错误信息表明，你尝试将 *sql.DB 类型的 conn 变量用作 DBTX 类型的参数，但 *sql.DB 并不满足 DBTX 接口的要求。具体来说，DBTX 接口期望的 Exec 方法签名是 (context.Context, string, ...interface{}) (pgconn.CommandTag, error)，而 *sql.DB 提供的 Exec 方法签名是 (string, ...any) (sql.Result, error)。
~~~



**目前不理解  方法和接口 开始学习**

~~~go
//这是接口
type DBTX interface {
	Exec(context.Context, string, ...interface{}) (pgconn.CommandTag, error)
	Query(context.Context, string, ...interface{}) (pgx.Rows, error)
	QueryRow(context.Context, string, ...interface{}) pgx.Row
}

string, ...interface{}//的含义这是可变参数的声明方式。它表示可以接收零个或多个任意类型的参数。在函数内部，这些参数被视为一个 []interface{}（一个接口类型的切片）。
例如，如果有一个函数接受一个字符串和任意数量的其他参数：

func myFunction(s string, args...interface{}) {
    // 函数体
}

可以这样调用这个函数：
myFunction("hello", 1, "world", true)




/*
 在这段代码中有一个这样的错误我不知道怎么修改cannot use conn (variable of type *sql.DB) as DBTX value in argument to New: *sql.DB does not implement DBTX (wrong type for method Exec) have Exec(string, ...any) (sql.Result, error) want Exec(context.Context, string, ...interface{}) (pgconn.CommandTag,



要解决这个问题，你需要确保你的 DBTX 接口的方法签名与 *sql.DB 的实现匹配。具体来说，你需要修改 DBTX 接口的 Exec 方法，使其接受 context.Context 参数。以下是一个可能的解决方案：

修改 DBTX 接口，使其 Exec 方法的签名为：

type DBTX interface {
    Exec(ctx context.Context, query string, args ...interface{}) (pgconn.CommandTag, error)
    // 其他方法...
}
在你的 New 函数中，确保它接受符合 DBTX 接口的类型。例如，使用 sql.DB 时，你可以创建一个适配器来实现 DBTX 接口。
这里是一个适配器示例：


type DBAdapter struct {
    *sql.DB
}

func (db *DBAdapter) Exec(ctx context.Context, query string, args ...interface{}) (pgconn.CommandTag, error) {
    result, err := db.DB.ExecContext(ctx, query, args...)
    return result, err
}
然后在 TestMain 中使用这个适配器：

testQueries = New(&DBAdapter{conn})
这样就能确保 *sql.DB 能正确实现 DBTX 接口，从而解决你遇到的错误。
*/

!!!!!!!!!!!!!!!!!!!!!!gpt给出解决问题的方法在我不知道用pgx的情况下 gpt给出用pgx连接
给gpt看完接口定义之后
由于你的 DBTX 接口使用了 pgx 库，你可以使用 pgx 的 Conn 类型而不是 *sql.DB。这里是一个简单的解决方案：
使用 pgx 连接：确保在代码中使用 pgx 的连接，而不是 *sql.DB。修改你的数据库连接部分：


import (
	"context"
	"log"
	"os"
	"testing"

	"github.com/jackc/pgx/v4"
)

const (
	dbDriver = "pgx"
	dbSource = "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable"
)

var testQueries *Queries

func TestMain(m *testing.M) {
	conn, err := pgx.Connect(context.Background(), dbSource)
	if err != nil {
		log.Fatal("cannot connect to db:", err)
	}
	testQueries = New(conn) // 确保 New 函数接收 pgx.Conn

	os.Exit(m.Run())
}
确保 New 函数接受 pgx.Conn：在你的 New 函数中，确保它接受 pgx.Conn 而不是 *sql.DB。
这样，你的 DBTX 接口与 pgx 的连接就能正常工作，避免了类型不匹配的问题。这样也简化了代码，直接使用了 pgx 连接而不需要适配器。



~~~

---



#### 10.8日 解决目前为止遇见的一个问题





*遇见的第一个大问题 在account_test.go中和main_test.go中出现的一些问题*

*和姐聊了一下 解决了 **testQueries = New(conn)**中的问题*

*main_test.go使用 pgx来连接数据库 而不是视频中讲解的sql.open 他们的返回值类型不同造成了错误*

测试函数的拼写错误 可能也影响了一大部分



还有最后一处问题

1. `require.NotEmpty(t, err)`这一行存在问题。这里应该是检查`account`是否不为空，而不是检查错误`err`是否不为空。正确的应该是`require.NotEmpty(t, account)`。



不熟悉的地方go语言的包 接口 方法

----



#### 1.最终的account_test.go代码

~~~go
package db

import (
	"context"
	"testing"

	"github.com/stretchr/testify/require"
)

func TestCreateAccount(t *testing.T) {
	arg := CreateAccountParams{
		Owner:    "xiaozhao",
		Balance:  100,
		Currency: "USD",
	}

	account, err := testQueries.CreateAccount(context.Background(), arg)
	require.NoError(t, err)
	require.NotEmpty(t, account)

	require.Equal(t, arg.Owner, account.Owner)
	require.Equal(t, arg.Balance, account.Balance)
	require.Equal(t, arg.Currency, account.Currency)

	require.NotZero(t, account.ID)
	require.NotZero(t, account.CreatedAt)
}

~~~



#### 2.最终的main_test.go代码

~~~go
package db

import (
	"context"
	"fmt"
	"os"
	"testing"

	"github.com/jackc/pgx/v5"
)

var testQueries *Queries

const (
	DATABASE_URL = "postgres://root:secret@localhost:5432/simple_bank?sslmode=disable"
)

func TestMain(m *testing.M) {

	conn, err := pgx.Connect(context.Background(), DATABASE_URL)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Unable to connect to database: %v\n", err)
		os.Exit(1)
	}
	defer conn.Close(context.Background())

	testQueries = New(conn)
	os.Exit(m.Run())
}

//测试通过！！！！！！！
~~~



**上面是指定了一个一个账户 我们想让账户的主人 货币 钱是随机的 编写util中的random代码**：

~~~go
package util

import (
    "math/rand"
    "strings"
    "time"
)

const alphabet = "abcdefghijklmnopqrstuvwxyz"

var rng *rand.Rand

func init() {
    source := rand.NewSource(time.Now().UnixNano())
    rng = rand.New(source)
}

// 返回一个介于 min max 之间的随机的 int64 数字
func RandomInt(min, max int64) int64 {
    return min + rng.Int63n(max-min+1)
}

// 生成 n 个字符的随机字符串
func RandomString(n int) string {
    var sb strings.Builder
    k := len(alphabet)

    for i := 0; i < n; i++ {
        c := alphabet[rng.Intn(k)]
        sb.WriteByte(c)
    }

    return sb.String()
}


//随机生成owner
func RandomOwner() string{
	return RandomString(6)
}

//随机生成钱的数量
func RandomMoney() int64{
	return RandomInt(0,1000 )
}

//随机产生一种货币
func RandowCurrency() string{
	currencies := []string{"RMB","USD","CAD"}
	n := len(currencies)
	return currencies[rand.Intn(n)]
}
~~~

**学会如何把自己写的包导入到别的文件夹下  这个需要看go mod下的 module project/simplebank  **

**把moudle中的包作为起始路径 导入到别的文件夹下  就是："project/simplebank/util"**



ok 截止到 10.8日随机生成的数据生成功

---



**问题2**

makefile文件中的下面这个指令

**test:**

**go test -v -cover ./…     这个指令必须在当前目录下找到go的测试文件**

**就是go.mod文件应该和makefile保持在一起**    解决方法在本地的go.mod文件夹下又创建了一个makefile 用来测试 make test



类型断言 interface代表为止类型 使用前需要 转换为具体类型   (从未知类型转为已知类型)

如：

~~~
var i interface{} = 2
num1， ok ：= i.(int)//断言
~~~

---



#### 3.account_test.go代码：

全部测试通过！

~~~go
package db

import (
	"context"

	"project/simplebank/util"
	"testing"
	"time"

	"github.com/stretchr/testify/require"
)

func createRandomAccount(t *testing.T) Account {
	arg := CreateAccountParams{
		Owner:    util.RandomOwner(),
		Balance:  util.RandomMoney(),
		Currency: util.RandomCurrency(),
	}

	account, err := testQueries.CreateAccount(context.Background(), arg)
	require.NoError(t, err)
	require.NotEmpty(t, account)

	require.Equal(t, arg.Owner, account.Owner)
	require.Equal(t, arg.Balance, account.Balance)
	require.Equal(t, arg.Currency, account.Currency)

	require.NotZero(t, account.ID)
	require.NotZero(t, account.CreatedAt)

	return account
}

func TestCreateAccount(t *testing.T) {

	createRandomAccount(t)
}

func TestGetAccount(t *testing.T) {
	account1 := createRandomAccount(t)
	account2, err := testQueries.GetAccount(context.Background(), account1.ID)

	require.NoError(t, err)
	require.NotEmpty(t, account2)

	require.Equal(t, account1.ID, account2.ID)
	require.Equal(t, account1.Owner, account2.Owner)
	require.Equal(t, account1.Balance, account2.Balance)
	require.Equal(t, account1.Currency, account2.Currency)

	require.WithinDuration(t, account1.CreatedAt.Time, account2.CreatedAt.Time, time.Second)
}

func TestUpdateAccount(t *testing.T) {
	account1 := createRandomAccount(t)

	arg := UpdateAccountParams{
		ID:      account1.ID,
		Balance: util.RandomMoney(),
	}
	account2, err := testQueries.UpdateAccount(context.Background(), arg)

	require.NoError(t, err)
	require.NotEmpty(t, account2)

	require.Equal(t, account1.ID, account2.ID)
	require.Equal(t, account1.Owner, account2.Owner)
	require.Equal(t, arg.Balance, account2.Balance)
	require.Equal(t, account1.Currency, account2.Currency)

	require.WithinDuration(t, account1.CreatedAt.Time, account2.CreatedAt.Time, time.Second)
}

func TestDeleteAccount(t *testing.T) {
	account1 := createRandomAccount(t)
	err := testQueries.DeleteAccount(context.Background(), account1.ID)
	require.NoError(t, err)

	account2, err := testQueries.GetAccount(context.Background(), account1.ID)
	require.Error(t, err)
	//	require.EqualError(t, err, sql.ErrNoRows.Error())
	require.Empty(t, account2)
}

func TestListAccount(t *testing.T) {
	var lastAccount Account
	for i := 0; i < 10; i++ {
		lastAccount = createRandomAccount(t)
	}
	arg := ListAccountsParams{
		Owner:  lastAccount.Owner,
		Limit:  5, //返回五条记录
		Offset: 0, //设置偏移量 返回后五条记录  这里出现了问题！！！！
	}
	accounts, err := testQueries.ListAccounts(context.Background(), arg)
	require.NoError(t, err)
	require.NotEmpty(t, accounts)

	for _, account := range accounts {
		require.NotEmpty(t, account)
		require.Equal(t, lastAccount.Owner, account.Owner)
	}
}

~~~

---



#### 4.entry_test.go

条目上的account.id要和account表单上的di相对应



**问题**



**id为null**

***在 PostgreSQL 中，如果一个表的 `id` 字段没有设置为自增序列（如 `bigserial`），并且你在插入数据时没有显式地为 `id` 字段指定值，那么 `id` 字段的值将会是 `NULL`，除非该字段设置了默认值。***

**解决办法**

~~~sql
创建一个序列：首先，你需要创建一个序列，这个序列将用于生成 id 列的值。
   CREATE SEQUENCE entries_id_seq;
~~~

~~~sql
设置序列的所有权：将序列与 id 列关联起来。
   ALTER SEQUENCE entries_id_seq OWNED BY entries.id;
~~~

~~~sql
设置 id 列的默认值为序列的下一个值：这样，每当你插入新行而没有指定 id 值时，PostgreSQL 会自动使用序列的下一个值。
   ALTER TABLE entries ALTER COLUMN id SET DEFAULT nextval('entries_id_seq');
 
~~~

**4确保 `id` 列是主键**：从你提供的信息来看，`id` 列已经是主键。确保这一点很重要，因为主键约束可以保证 `id` 列的值是唯一的。

~~~sql
测试：插入一条新记录，不指定 id 值，检查是否自动生成了 id。
   INSERT INTO entries (account_id, amount, created_at) VALUES (1, 100, now());
~~~

~~~sql
DELETE FROM entries WHERE id=4; 删除特定行的指令
~~~



创建账单成功！



但是只能生成一个数据？？？

我发现了输出的区别 Running tool: D:\Go\bin\go.exe test -timeout 30s -run ^TestCreateEntry$ project/simplebank/db/sqlc

ok  	project/simplebank/db/sqlc	(cached) 这是第二次输出        第一次输出没有cached字样 数据正确的加载到了数据库 但是这个带有cached的数据没有加载到数据库



**因为 cached 是因为两次的数据相同 所以才没有被加载到数据库 这个可能是随机数代码的问题**

---

#### 5.transfer_test.go

~~~go
package db

import (
	"context"
	"time"

	"project/simplebank/util"
	"testing"

	"github.com/stretchr/testify/require"
)

func createRandomTransfer(t *testing.T, account1, account2 Account) Transfer {
	arg := createTransferParams{
		FromAccountID: account1.ID,
		ToAccountID:   account2.ID,
		Amount:        util.RandomMoney(),
	}
	transfer, err := testQueries.createTransfer(context.Background(), arg)
	require.NoError(t, err)
	require.NotEmpty(t, transfer)

	require.Equal(t, arg.FromAccountID, transfer.FromAccountID)
	require.Equal(t, arg.ToAccountID, transfer.ToAccountID)
	require.Equal(t, arg.Amount, transfer.Amount)

	require.NotZero(t, transfer.ID)
	require.NotZero(t, transfer.CreatedAt)

	return transfer
}

func TestCreateTransfer(t *testing.T) {
	account1 := createRandomAccount(t)
	account2 := createRandomAccount(t)
	createRandomTransfer(t, account1, account2)
}

func TestGetTransfer(t *testing.T) {
	account1 := createRandomAccount(t)
	account2 := createRandomAccount(t)
	transfer1 := createRandomTransfer(t, account1, account2)

	transfer2, err := testQueries.GetTransfer(context.Background(), transfer1.ID)
	require.NoError(t, err)
	require.NotEmpty(t, transfer2)

	require.Equal(t, transfer1.ID, transfer2.ID)
	require.Equal(t, transfer1.FromAccountID, transfer2.FromAccountID)
	require.Equal(t, transfer1.ToAccountID, transfer2.ToAccountID)
	require.Equal(t, transfer1.Amount, transfer2.Amount)
	require.WithinDuration(t, transfer1.CreatedAt.Time, transfer2.CreatedAt.Time, time.Second)
}

func TestListTransfer(t *testing.T) {
	account1 := createRandomAccount(t)
	account2 := createRandomAccount(t)

	for i := 0; i < 5; i++ {
		createRandomTransfer(t, account1, account2)
		createRandomTransfer(t, account2, account1)
	}

	arg := ListTransfersParams{
		FromAccountID: account1.ID,
		ToAccountID:   account1.ID,
		Limit:         5,
		Offset:        5,
	}

	transfers, err := testQueries.ListTransfers(context.Background(), arg)
	require.NoError(t, err)
	require.Len(t, transfers, 5)

	for _, transfer := range transfers {
		require.NotEmpty(t, transfer)
		require.True(t, transfer.FromAccountID == account1.ID || transfer.ToAccountID == account1.ID)
	}
}

~~~



### 十.db transaction



BEGIN语句启动事务

成功 则更新数据库

失败 则回滚事务（保持原来的状态）

​

代码对不上了 决定先复制粘贴 学习数据库中的知识点





**先从config.go开始**

****

### 十一.config.go

**使用viper**

创建app.env文件存储配置信息

config.go

~~~go
package util

import (
	"github.com/spf13/viper"
)

// Config stores all configuration of the application.
// The values are read by viper from a config file or environment variable.
type Config struct {
	DATABASE_URL string `mapstructure:"DATABASE_URL"`
}

// LoadConfig reads configuration from file or environment variables.
func LoadConfig(path string) (config Config, err error) {
	viper.AddConfigPath(path)
	viper.SetConfigName("app")
	viper.SetConfigType("env")

	viper.AutomaticEnv()

	err = viper.ReadInConfig()
	if err != nil {
		return
	}

	err = viper.Unmarshal(&config)
	return
}
~~~

---



使用接口来简化一些操作好好学接口

目前为止更正了大部分问题接着往下学。。。。

**store.test.go出现了大问题**

**报错：**

**Running tool: D:\Go\bin\go.exe test -timeout 30s -run ^TestTransferTx$ project/simplebank/db/sqlc >> before: 1984 3906 --- FAIL: TestTransferTx (0.03s)    e:\projects\simplebank\db\sqlc\store_test.go:83:         	Error Trace:	e:/projects/simplebank/db/sqlc/store_test.go:83        	Error:      	Should NOT be empty, but was {0  0  {0001-01-01 00:00:00 +0000 UTC finite false}}        	Test:       	TestTransferTx FAIL FAIL	project/simplebank/db/sqlc	****0.571s**



因为还没编写代码。。。。。草草草草操操操操哦哦操操操这视频叫我看的

---

### 十二.需要仔细处理并发 交易 以避免死锁

数据库事务

~~~shell
Running tool: D:\Go\bin\go.exe test -timeout 30s -run ^TestTransferTx$ project/simplebank/db/sqlc

>> before: 6892 6969
>> tx: 6882 6979
>> tx: 6882 6989
--- FAIL: TestTransferTx (0.04s)
    e:\projects\simplebank\db\sqlc\store_test.go:102: 
        	Error Trace:	e:/projects/simplebank/db/sqlc/store_test.go:102
        	Error:      	Not equal: 
        	            	expected: 10
        	            	actual  : 20
        	Test:       	TestTransferTx
FAIL
FAIL	project/simplebank/db/sqlc	0.602s
FAIL

~~~

这个问题出在

account.sql.go他无法阻止一些东西

~~~
-- name: GetAccount :one
SELECT * FROM accounts
WHERE id = $1 LIMIT 1;
~~~

**在两个终端中并行运行两个事务来观察这个问题**

*BEGIN；开始事务*

*ROLLBACK；回滚事务*

：第一个终端

~~~~shell
simple_bank=# BEGIN;
BEGIN
simple_bank=# BEGIN;
WARNING:  there is already a transaction in progress
BEGIN
simple_bank=# ROLLBACK;
ROLLBACK
simple_bank=# BEGIN;
BEGIN
simple_bank=# SELECT * FROM accounts WHERE id=1 FOR UPDATE;
 id |  owner   | balance | currency |         created_at
----+----------+---------+----------+----------------------------
  1 | xiaozhao |     100 | USD      | 2024-10-08 09:03:03.272176
(1 row)
~~~~

第二个终端

~~~shell
simple_bank=# BEGIN;
BEGIN
simple_bank=# SELECT * FROM accounts WHERE id=1 FOR UPDATE;

这里会被阻止 并且必须等待第一个事务提交或回滚
~~~



纠正方法1： 在sql中添加 ： 重新用make sqlc生成

~~~sql
-- name: GetAccountForUpdate :one
SELECT * FROM accounts
WHERE id = $1 LIMIT 1
FOR UPDATE;
~~~

但是接下来出现了死锁错误：

添加日志寻找错误

~~~
Running tool: D:\Go\bin\go.exe test -timeout 30s -run ^TestTransferTx$ project/simplebank/db/sqlc

>> before: 1826 5993
tx 5 Create transfer
tx 5 Create entry 1
tx 5 Create entry 2
tx 5 get account 1
tx 2 Create transfer
tx 5 update account 1
tx 5 get account 2
tx 4 Create transfer
tx 5 update account 2
tx 3 Create transfer
tx 2 Create entry 1
tx 4 Create entry 1
tx 3 Create entry 1
tx 1 Create transfer
tx 2 Create entry 2
tx 4 Create entry 2
tx 3 Create entry 2
tx 2 get account 1
tx 4 get account 1
tx 3 get account 1
>> tx: 1816 6003
tx 1 Create entry 1
tx 1 Create entry 2
tx 1 get account 1
--- FAIL: TestTransferTx (0.95s)
    e:\projects\simplebank\db\sqlc\store_test.go:52: 
        	Error Trace:	e:/projects/simplebank/db/sqlc/store_test.go:52
        	Error:      	Received unexpected error:
        	            	ERROR: deadlock detected (SQLSTATE 40P01)
        	Test:       	TestTransferTx
FAIL
FAIL	project/simplebank/db/sqlc	1.470s
FAIL

~~~



终端事务出现错误：

~~~sql
INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1,2,10) RETURNING *;
~~~



1. - 第一个错误 `INSERT INFO transfers` 是语法错误，正确的语法是 `INSERT INTO transfers`。
- 第二个错误 `INSERT INTO transfers (from_account_id to_account_id amount)` 也存在语法错误，缺少逗号分隔列名。正确的写法是 `INSERT INTO transfers (from_account_id, to_account_id, amount)`。
2. **当前事务已中止**：
    - 由于之前的 SQL 语句（可能是第一条插入语句）出错，事务被标记为 "aborted"。这意味着在该事务中的所有后续 SQL 命令都将失败，直到事务被回滚。

### 解决方法

1. **结束当前事务**：

    - 在 PostgreSQL 中，你可以通过以下命令结束当前事务并回滚更改：

      ~~~sql
      ROLLBACK
      ~~~

#### 终端阻塞 事务状态



~~~
1. 确认当前事务状态
在 PostgreSQL 中，如果一个事务因为某种原因（例如错误或未处理的异常）而中断，那么所有后续的 SQL 语句将会被忽略，直到你执行 ROLLBACK 或 COMMIT。首先，确保没有事务在进行中。

你可以使用以下命令查看当前活动的事务：

SELECT * FROM pg_stat_activity WHERE state = 'active';
~~~

~~~sql
simple_bank=# INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1,2,10) RETURNING *;
^CCancel request sent
ERROR:  canceling statement due to user request
CONTEXT:  SQL statement "SELECT 1 FROM ONLY "public"."accounts" x WHERE "id" OPERATOR(pg_catalog.=) $1 FOR KEY SHARE OF x"
simple_bank=# SELECT * FROM pg_stat_activity WHERE state = 'active';
 datid |   datname   | pid | usesysid | usename | application_name | client_addr | client_hostname | client_port |         backend_start         |          xact_start           |          query_start          |         state_change
 | wait_event_type |  wait_event   | state  | backend_xid | backend_xmin |                                            query                                            |  backend_type
-------+-------------+-----+----------+---------+------------------+-------------+-----------------+-------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------+-----------------+---------------+--------+-------------+--------------+---------------------------------------------------------------------------------------------+----------------
 16385 | simple_bank | 810 |       10 | root    | psql             |             |                 |          -1 | 2024-10-10 12:58:51.115963+00 | 2024-10-10 13:00:09.094842+00 | 2024-10-10 13:00:09.094842+00 | 2024-10-10 13:00:09.094867+00 |                 |               | active |             |          935 | SELECT * FROM pg_stat_activity WHERE state = 'active';                                      | client backend
 16385 | simple_bank | 802 |       10 | root    | psql             |             |                 |          -1 | 2024-10-10 12:58:07.854964+00 | 2024-10-10 12:58:09.850035+00 | 2024-10-10 12:58:09.850035+00 | 2024-10-10 12:58:09.850039+00 | Lock            | tuple         | active |         965 |          935 | INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1,2,10) RETURNING *; | client backend
 16385 | simple_bank | 782 |       10 | root    | psql             |             |                 |          -1 | 2024-10-10 12:51:58.685461+00 | 2024-10-10 12:57:47.014409+00 | 2024-10-10 12:57:47.014409+00 | 2024-10-10 12:57:47.014413+00 | Lock            | tuple         | active |         963 |          935 | INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1,2,10) RETURNING *; | client backend
 16385 | simple_bank | 775 |       10 | root    | psql             |             |                 |          -1 | 2024-10-10 12:51:51.971697+00 | 2024-10-10 12:52:16.478898+00 | 2024-10-10 12:57:54.280622+00 | 2024-10-10 12:57:54.280627+00 | Lock            | tuple         | active |         964 |          935 | INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1,2,10) RETURNING *; | client backend
 16385 | simple_bank | 556 |       10 | root    | psql             |             |                 |          -1 | 2024-10-10 11:29:44.776508+00 | 2024-10-10 11:56:11.909182+00 | 2024-10-10 11:56:24.551023+00 | 2024-10-10 11:56:24.551027+00 | Lock            | transactionid | active |             |          935 | SELECT * FROM accounts WHERE id=1 FOR UPDATE;                                               | client backend
 16385 | simple_bank | 638 |       10 | root    | psql          
~~~

---

从你提供的 `pg_stat_activity` 的查询结果来看，有多个活跃的事务正在等待锁。这可能是导致你无法执行 `INSERT` 操作的原因。以下是一些步骤，可以帮助你诊断并解决这个问题：

#### 1. **识别并终止阻塞的事务**

你的查询结果显示了多个 `INSERT` 操作在等待锁。可以尝试强制终止这些阻塞的事务，以释放锁并允许你的操作继续进行。首先，记录下阻塞的 `pid`（进程 ID），然后使用以下命令终止它们：

```
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid IN (802, 782, 775);
```

在这个例子中，`802`、`782` 和 `775` 是阻塞的 `pid`。请根据你的实际情况调整这条命令。

#### 2. **确认事务的状态**

如果终止了这些连接，确保你没有其他活跃的事务在运行。你可以再次运行以下命令确认：

```
SELECT * FROM pg_stat_activity WHERE state = 'active';
```

#### 3. **尝试再次执行插入**

在确保没有其他事务阻塞后，尝试重新执行你的插入操作：

```sql
BEGIN;  -- 开始新的事务
INSERT INTO transfers (from_account_id, to_account_id, amount) VALUES (1, 2, 10) RETURNING *;  -- 执行插入
COMMIT;  -- 提交事务

指令；
SELECT * FROM accounts WHERE id=1 FOR UPDATE;
UPDATE accounts SET balance = 600 WHERE id=1;
```

#### 4. **如果问题依然存在**

如果问题依然存在，建议检查数据库的锁状态：

```
SELECT * FROM pg_locks WHERE granted = false;
```

**又产生错误**

~~~
simple_bank=# SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid IN (802, 782, 775,556,638,810);
FATAL:  terminating connection due to administrator command
server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
~~~

回答：

看起来你在尝试终止阻塞的连接时，PostgreSQL 服务器意外关闭了连接。这可能是由于以下几个原因：

1. **服务器崩溃**：可能是因为在终止连接时，某个后台进程遇到了问题，导致服务器崩溃。
2. **资源不足**：如果你的 PostgreSQL 服务器资源不足（例如内存或 CPU），在处理请求时可能会出现这种情况。

~~~
由于服务器崩溃，重启 Docker 容器可能会帮助你恢复连接：

docker restart postgres12
在重启后检查连接：

重启后，尝试重新连接到数据库，并检查活动连接：

SELECT * FROM pg_stat_activity;
再次终止阻塞的连接：

如果连接正常，尝试再次运行终止命令：

SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid IN (802, 782, 775, 
~~~



---



**git上传一个项目没有共同历史**

~~~shell
检查是否有共同历史
git log --oneline --graph --all
~~~

~~~shell
强制合并冲突
git pull origin main --allow-unrelated-histories
~~~

---



终端1：在没有阻塞的情况下

~~~sql
simple_bank=# BEGIN;
BEGIN
simple_bank=# SELECT * FROM accounts WHERE id=1 FOR UPDATE;
 id |  owner   | balance | currency |         created_at
----+----------+---------+----------+----------------------------
  1 | xiaozhao |     100 | USD      | 2024-10-08 09:03:03.272176
(1 row)

simple_bank=# UPDATE accounts SET balance = 600 WHERE id=1;
UPDATE 1
simple_bank=# COMMIT;
COMMIT
simple_bank=#

~~~

在终端一提交事务时 终端二会显示出结果

~~~sql
simple_bank=# BEGIN;
BEGIN
simple_bank=# SELECT * FROM accounts WHERE id=1 FOR UPDATE;
 id |  owner   | balance | currency |         created_at
----+----------+---------+----------+----------------------------
  1 | xiaozhao |     600 | USD      | 2024-10-08 09:03:03.272176
(1 row)

simple_bank=#


~~~



sql QUERIER

~~~sql
BEGIN;

INSERT INTO transfers (from_account_id,to_account_id,amount) VALUE (1,2,10) RETURNING *;

INSERT INTO entries (account_id,amount) VALUES (1,-10) RETURNING *;
INSERT INTO entries (account_id,amount) VALUES (2,10) RETURNING *;

SELECT * FROM accounts WHERE id=1 FOR UPDATE;
UPDATE accounts SET balance = 90 WHERE id = 1 RETURNING *;

SELECT * FROM accounts WHERE id =2 FOR UPDATE;
UPDATE accounts SET balance = 110 WHERE id = 2 RETURNING *;

ROLLBACK;
~~~

---



#### 5.postgres lock：帮助查询哪里有锁

The following query may be helpful to see what processes are blocking SQL statements (these only find row-level locks, not object-level locks).

```
  SELECT blocked_locks.pid     AS blocked_pid,
         blocked_activity.usename  AS blocked_user,
         blocking_locks.pid     AS blocking_pid,
         blocking_activity.usename AS blocking_user,
         blocked_activity.query    AS blocked_statement,
         blocking_activity.query   AS current_statement_in_blocking_process
   FROM  pg_catalog.pg_locks         blocked_locks
    JOIN pg_catalog.pg_stat_activity blocked_activity  ON blocked_activity.pid = blocked_locks.pid
    JOIN pg_catalog.pg_locks         blocking_locks 
        ON blocking_locks.locktype = blocked_locks.locktype
        AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
        AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
        AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
        AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
        AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
        AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
        AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
        AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
        AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
        AND blocking_locks.pid != blocked_locks.pid

    JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
   WHERE NOT blocked_locks.granted;
```

~~~sql
SELECT * FROM accounts WHERE id=1 FOR UPDATE; 这条语句阻塞了
~~~

~~~sql
Here's an alternate view of that same data that includes an idea how old the state is
# 列出所有锁

SELECT a.datname,
         l.relation::regclass,
         l.transactionid, //事务id
         l.mode, 锁的mod
         l.GRANTED,
         a.usename,  who
         a.query, 
         a.query_start,
         age(now(), a.query_start) AS "age",
         a.pid
FROM pg_stat_activity a
JOIN pg_locks l ON l.pid = a.pid
ORDER BY a.query_start;
~~~



**死锁是由外键约束引起的**

1.删除约束

修改sql

~~~sql
-- name: GetAccountForUpdate :one
SELECT * FROM accounts
WHERE id = $1 LIMIT 1
FOR NO KEY UPDATE;//这步时解决死锁的关键
~~~

避免死锁是关键：微调事务中的查询



### 十三.隔离级别

数据库事务必须满足 ACID 原子性  一致性 隔离性 持久性

----



#### Read Phenomenaa



#### 1.脏读

当一个事务读取了   其他并发事务写入的尚未提交的数据（导致 如果尚未提交的数据 最终回滚 可能导致用到错误的数据 ）

#### 2.不可重复读

当一个事务两次读取到同一记录并看到不同的值  因为第一次读取后提交的其他事务修改

#### 3.幻读

影响多行

#### 4.四种隔离级别

READ UNCOMMITMED： 可以看到其他未提交事务写入的数据

READ COMMITED：只能看到其他事务已经提交的数据

REPEATABLE READ:

SERIALIZABLE:



#### 5.mysql选择隔离级别

~~~sql
set sexxion transaction isolation level read commited;

select @@一种隔离级别
~~~

#### 6.postgresql选择隔离级别 只有三个

~~~
在postgresql中 未提交和已提交是一个级别
show transaction isolation level

set transaction isolation level read uncommited

~~~

### 十四.持续集成或CI



**自动化构建和测试流程进行验证**

#### 1.Github Action





首先上传项目到github时如果出现了连接问题 就切换成ssh连接

~~~shell
git remote set-url origin git@github.com:Whuichenggong/projects.git
PS E:\projects> git pull origin main --tags
From github.com:Whuichenggong/projects
 * branch            main       -> FETCH_HEAD
~~~



创建文件

~~~shell
echo. > .github\workflows\ci.yml
~~~

安装工具

~~~
golang migrate
~~~



但是目前我看不到页面我的action



### 十五.RESTful HEEP API

#### 1.创建api文件夹

**account.go**

~~~go
package api

import (
	"net/http"
	db "project/simplebank/db/sqlc"

	"github.com/gin-gonic/gin"
)

type CreateAccountRequest struct {
	Owner    string `json:"owner" binding:"required"`
	Currency string `json:"currency" binding:"required,oneof= USD EUR"`
}

func (server *Server) createAccount(ctx *gin.Context) {
	var req CreateAccountRequest
	if err := ctx.ShouldBindJSON(&req); err != nil {
		ctx.JSON(http.StatusBadRequest, errorResponse(err))
		return
	}
	arg := db.CreateAccountParams{
		Owner:    req.Owner,
		Currency: req.Currency,
		Balance:  0,
	}
	account, err := server.store.CreateAccount(ctx, arg)
	if err != nil {
		ctx.JSON(http.StatusInternalServerError, errorResponse(err))
		return
	}

	ctx.JSON(http.StatusOK, account)
}

~~~



**server.go**

~~~go
package api

import (
	db "project/simplebank/db/sqlc"
	"project/simplebank/util"

	"github.com/gin-gonic/gin"
)

type Server struct {
	config util.Config
	store  db.Store
	router *gin.Engine
}

func NewServer(config util.Config, store db.Store) (*Server, error) {
	server := &Server{
		config: config,
		store: store,
	}
	router := gin.Default()

	router.POST("/accounts", server.createAccount)

	server.router = router
	return server, nil
}

func errorResponse(err error) gin.H {
	return gin.H{"error": err.Error()}
}

func (server *Server) Start(address string) error {
	return server.router.Run(address)
}

~~~

**main.go**

~~~go
package main

import (
	"context"
	"log"
	"project/simplebank/api"
	"project/simplebank/util"

	db "project/simplebank/db/sqlc"

	"github.com/jackc/pgx/v5/pgxpool"
)

func main() {
	config, err := util.LoadConfig(".")
	if err != nil {
		log.Fatal("cannot load config:", err)
	}

	connPool, err := pgxpool.New(context.Background(), config.DATABASE_URL)
	if err != nil {
		log.Fatal("cannot connect to db:", err)
	}
	//初始化数据库服务
	store := db.NewStore(connPool)
	//运行gin框架
	runGinServer(config, store)
	

	if err != nil {
		log.Fatal("cannot start server:", err)
	}

}

func runGinServer(config util.Config, store db.Store) {
	server, err := api.NewServer(config, store)
	if err != nil {
		log.Fatalf("cannot create server: %v", err)
	}
	err = server.Start(config.HTTPServerAddress)
	if err != nil {
		log.Fatalf("cannot start server: %v", err)
	}
}

~~~



**数据库重置**

~~~sql
MySQL 数据库：
使用 TRUNCATE TABLE 语句：

   TRUNCATE TABLE table_name;
   
PostgreSQL 数据库：
使用 TRUNCATE TABLE 语句：

   TRUNCATE TABLE table_name RESTART IDENTITY;
   
   TRUNCATE TABLE accounts, entries RESTART IDENTITY; 同时截断两个表
~~~



**listaccount.go**

用postman请求时：//查询参数

**page_id     1**

**page_size   5**

**在使用多组查找的时候没有找到用户？？？**

目前为止还是无法解决

-----

找了喜春学哥帮我找到了问题的所在在ListAccounts中 传进去的arg.Owner是个空值导致了出现了问题 把arg.Owner改成一个数据库中具体的值 就能找到问题的所在

~~~go
func (q *Queries) ListAccounts(ctx context.Context, arg ListAccountsParams) ([]Account, error) {
	rows, err := q.db.Query(ctx, listAccounts, arg.Owner/*问题所在*/, arg.Limit, arg.Offset)
~~~



---

#### 2.模拟数据库测试

~~~shell
使用mock
 go get github.com/golang/mock/mockgen@v1.6.0
 
PS E:\projects\simplebank\db\mock> mockgen -destination db/mock/store.go project/simplebank/db/sqlc Store 
~~~

#### 3.account_test.go



出现的问题

----

你提到的问题是由于 `mock_sqlc.MockStore` 未完全实现 `db.Store` 接口，特别是缺少 `createTransfer` 方法。为了解决这个问题，您可以采取以下步骤：

解决步骤：

1. **确认 `db.Store` 接口的定义：**

首先，确保 `db.Store` 接口定义了所有需要的方法。特别是，确认接口中是否包含 `createTransfer` 方法。

也就是：

~~~
在你当前的测试代码中，store := mockdb.NewMockStore(ctrl) 返回的确实是 *mockdb.MockStore 类型，而 NewServer 需要的参数是 db.Store 接口类型。那么为什么没有类型错误呢？这是因为在 Go 中，接口是基于方法集实现的，而 *mockdb.MockStore 实现了 db.Store 接口中的所有方法。

具体原因分析：
接口实现方式：在 Go 语言中，接口并不关心你传递的具体类型（如 *mockdb.MockStore），它只关心该类型是否实现了接口中定义的所有方法。如果 *mockdb.MockStore 实现了 db.Store 接口的所有方法，那么它就可以被赋值给 db.Store 类型的变量。

gomock 的自动生成：你使用 gomock 生成了 *mockdb.MockStore，这个 mock 类型会模拟 db.Store 接口的所有方法。因为它是通过 gomock 自动生成的，并且已经包含了 db.Store 中的所有方法，所以它实际上是符合 db.Store 接口的实现。

类型匹配：在 Go 中，赋值 *mockdb.MockStore 给 db.Store 类型是可以的，因为 *mockdb.MockStore 实现了 db.Store 接口。即便 *mockdb.MockStore 是一个具体类型，只要它的方法集与 db.Store 接口的方法集匹配，Go 会认为它是一个合法的接口实现。

为什么没有错误？
由于 *mockdb.MockStore 实现了 db.Store 接口的所有方法，Go 编译器允许将 *mockdb.MockStore 传递给 NewServer 这个需要 db.Store 类型的函数参数。具体的原因是：

NewMockStore 生成的 mock 类型实现了 db.Store 的所有方法，因此符合 db.Store 接口。
在 Go 语言中，接口实现是隐式的，不需要显式声明实现接口，只要结构体的方法集与接口匹配即可。
~~~

方法 1：使用类型断言验证

在 Go 中，你可以通过**静态类型检查**来验证一个类型是否实现了某个接口。具体方法是使用以下代码：

```
var _ db.Store = (*mockdb.MockStore)(nil)
```

---





~~~go
package api

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"net/http/httptest"
	db "project/simplebank/db/sqlc"
	"project/simplebank/util"
	"testing"

	mockdb "project/simplebank/db/mock"

	"github.com/golang/mock/gomock"
	"github.com/stretchr/testify/require"
)

func TestGetAccountAPI(t *testing.T) {
	config, err := util.LoadConfig(".")
	if err != nil {
		fmt.Println("配置文件出错")
	}
	//user, _ := randomUser(t)
	account := randomAccount()
	testCases := []struct {
		name      string
		accountID int64
		//setupAuth     func(t *testing.T, request *http.Request, tokenMaker token.Maker)
		buildStubs    func(store *mockdb.MockStore)
		checkResponse func(t *testing.T, recoder *httptest.ResponseRecorder)
	}{
		{
			name:      "OK",
			accountID: account.ID,
			// setupAuth: func(t *testing.T, request *http.Request, tokenMaker token.Maker) {
			// 	addAuthorization(t, request, tokenMaker, authorizationTypeBearer, user.Username, user.Role, time.Minute)
			// },
			buildStubs: func(store *mockdb.MockStore) {
				store.EXPECT().
					GetAccount(gomock.Any(), gomock.Eq(account.ID)).
					Times(1).
					Return(account, nil)
			},
			checkResponse: func(t *testing.T, recorder *httptest.ResponseRecorder) {
				require.Equal(t, http.StatusOK, recorder.Code)
				requireBodyMatchAccount(t, recorder.Body, account)
			},
		},
	}

	for i := range testCases {
		tc := testCases[i]

		t.Run(tc.name, func(t *testing.T) {

			ctrl := gomock.NewController(t)
			defer ctrl.Finish()

			store := mockdb.NewMockStore(ctrl)
			tc.buildStubs(store)

			server, _ := NewServer(config, store)

			recorder := httptest.NewRecorder()

			url := fmt.Sprintf("/accounts/%d", tc.accountID)
			request, err := http.NewRequest(http.MethodGet, url, nil)
			require.NoError(t, err)

			//tc.setupAuth(t, request, server.tokenMaker)
			server.router.ServeHTTP(recorder, request)
			tc.checkResponse(t, recorder)
		})
	}
}

func randomAccount() db.Account {
	return db.Account{
		ID: util.RandomInt(1, 1000),
		//Owner:    owner,
		Balance:  util.RandomMoney(),
		Currency: util.RandomCurrency(),
	}
}

func requireBodyMatchAccount(t *testing.T, body *bytes.Buffer, account db.Account) {
	data, err := io.ReadAll(body)
	require.NoError(t, err)

	var gotAccount db.Account
	err = json.Unmarshal(data, &gotAccount)
	require.NoError(t, err)
	require.Equal(t, account, gotAccount)
}

~~~

**在这段代码中有不懂的地方**

~~~go
store := mockdb.NewMockStore(ctrl)
			tc.buildStubs(store)

			server, _ := NewServer(config, store)

store是 *mockdb.MockStore类型
而func NewServer(config util.Config, store db.Store) (*Server, error) 需要的是db.store类型
server, _ := NewServer(config, store)
//我觉得这是自相矛盾
~~~

**切片**

~~~go
testCases := []struct {
		name      string
		accountID int64
		//setupAuth     func(t *testing.T, request *http.Request, tokenMaker token.Maker)
		buildStubs    func(store *mockdb.MockStore)
		checkResponse func(t *testing.T, recoder *httptest.ResponseRecorder)
	}{
		{
			name:      "OK",
			accountID: account.ID,
			// setupAuth: func(t *testing.T, request *http.Request, tokenMaker token.Maker) {
			// 	addAuthorization(t, request, tokenMaker, authorizationTypeBearer, user.Username, user.Role, time.Minute)
			// },
			buildStubs: func(store *mockdb.MockStore) {
				store.EXPECT().
					GetAccount(gomock.Any(), gomock.Eq(account.ID)).
					Times(1).
					Return(account, nil)
			},
			checkResponse: func(t *testing.T, recorder *httptest.ResponseRecorder) {
				require.Equal(t, http.StatusOK, recorder.Code)
				requireBodyMatchAccount(t, recorder.Body, account)
			},
		},
		{
			name:      "NotFound",
			accountID: account.ID,
			buildStubs: func(store *mockdb.MockStore) {
				store.EXPECT().
					GetAccount(gomock.Any(), gomock.Eq(account.ID)).
					Times(1).
					Return(db.Account{}, sql.ErrNoRows)
			},
			checkResponse: func(t *testing.T, recorder *httptest.ResponseRecorder) {
				require.Equal(t, http.StatusNotFound, recorder.Code)

			},
		},
	}
~~~



----

目前的问题是notfound处理不符合预期

~~~go
{
			name:      "NotFound",
			accountID: account.ID,
			setupAuth: func(t *testing.T, request *http.Request, tokenMaker token.Maker) {
				addAuthorization(t, request, tokenMaker, authorizationTypeBearer, user.Username, user.Role, time.Minute)
			},

			buildStubs: func(store *mockdb.MockStore) {
				store.EXPECT().
					GetAccount(gomock.Any(), gomock.Eq(account.ID)).
					Times(1).
					Return(db.Account{}, db.ErrRecordNotFound)
			},
			checkResponse: func(t *testing.T, recorder *httptest.ResponseRecorder) {
				require.Equal(t, http.StatusNotFound, recorder.Code)//我手动把 recorder.Code换成404jiu'cheng'gogn
			},
		},
~~~



----

#### 4.transfer.go



Currency      string `json:"currency" binding:"required,currency"` 添加了currency验证器 因为正常json不能识别USD等货币

实现思路 在go run mian.go后使用gin框架请求路由前 使用自己添加的数字验证器

在server.go中添加如下内容

~~~go
// 自定义验证函数，检查 currency 是否为 "USD"
func validCurrency(fl validator.FieldLevel) bool {
	currency := fl.Field().String()
	return currency == "USD"
}

// 注册自定义验证器
func (server *Server) setupValidator() {
	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
		v.RegisterValidation("currency", validCurrency)
	}
}

func NewServer(config util.Config, store db.Store) (*Server, error) {
	server := &Server{
		config: config,
		store:  store,
	}

	// 注册自定义验证器
	server.setupValidator()
	router := gin.Default()

	router.POST("/accounts", server.createAccount)
	router.GET("/accounts/:id", server.getAccount)
	router.GET("/accounts", server.listAccount)

	router.POST("transfers", server.createTransfer)
	server.router = router
	return server, nil
}

func errorResponse(err error) gin.H {
	return gin.H{"error": err.Error()}
}

func (server *Server) Start(address string) error {
	return server.router.Run(address)
}

~~~





~~~go
package api

import (
	"errors"
	"fmt"
	"net/http"
	db "project/simplebank/db/sqlc"

	"github.com/gin-gonic/gin"
)

type transferRequest struct {
	FromAccountID int64  `json:"from_account" binding:"required,min=1"`
	ToAccountID   int64  `json:"to_account" binding:"required,min=1"`
	Amount        int64  `json:"amount" binding:"required,gt=0"`
	Currency      string `json:"currency" binding:"required,currency"`
}

func (server *Server) createTransfer(ctx *gin.Context) {

	var req transferRequest

	if err := ctx.ShouldBindJSON(&req); err != nil {
		ctx.JSON(http.StatusBadRequest, errorResponse(err))
		return
	}

	// 获取并处理 FromAccount
	fromAccount, valid := server.validAccount(ctx, req.FromAccountID, req.Currency)
	if !valid {
		return
	}

	// 获取并处理 ToAccount
	toAccount, valid := server.validAccount(ctx, req.ToAccountID, req.Currency)
	if !valid {
		return
	}

	arg := db.TransferTxParams{
		FromAccountID: fromAccount.ID,
		ToAccountID:   toAccount.ID,
		Amount:        req.Amount,
	}
	result, err := server.store.TransferTx(ctx, arg)
	if err != nil {
		ctx.JSON(http.StatusInternalServerError, errorResponse(err))
		return
	}

	ctx.JSON(http.StatusOK, result)
}

// 检查id和货币
func (server *Server) validAccount(ctx *gin.Context, accountID int64, currency string) (db.Account, bool) {
	account, err := server.store.GetAccount(ctx, accountID)
	if err != nil {
		if errors.Is(err, db.ErrRecordNotFound) {
			ctx.JSON(http.StatusNotFound, errorResponse(err))
			return account, false
		}

		ctx.JSON(http.StatusInternalServerError, errorResponse(err))
		return account, false
	}

	if account.Currency != currency {
		err := fmt.Errorf("account [%d] currency mismatch: %s vs %s", account.ID, account.Currency, currency)
		ctx.JSON(http.StatusBadRequest, errorResponse(err))
		return account, false
	}

	return account, true
}

~~~



用postman测试得到的内容

~~~json
{
    "transfer": {
        "id": 35,
        "from_account_id": 3,
        "to_account_id": 5,
        "amount": 12,
        "created_at": "2024-10-20T04:53:01.433988Z"
    },
    "from_account": {
        "id": 3,
        "owner": "afmxtl",
        "balance": 103,
        "currency": "USD",
        "created_at": "2024-10-13T13:33:43.423875Z"
    },
    "to_account": {
        "id": 5,
        "owner": "bdupue",
        "balance": 119,
        "currency": "USD",
        "created_at": "2024-10-13T13:37:04.113466Z"
    },
    "from_entry": {
        "id": 45,
        "account_id": 3,
        "amount": -12,
        "created_at": "2024-10-20T04:53:01.433988Z"
    },
    "to_entry": {
        "id": 46,
        "account_id": 5,
        "amount": 12,
        "created_at": "2024-10-20T04:53:01.433988Z"
    }
}
~~~

### 十六.用户身份验证和授权

#### 1.建user数据库表



~~~sql
// Use DBML to define your database structure
// Docs: https://dbml.dbdiagram.io/docs
Table user as U{
  username carchar [pk]
  hashed_paassword varchar [not null]
  full_name varchar [not null]
  email varchar [unique, not null]
  password_changed_at timestamp [not null, default: `0001-01-01 00:00:00Z`]
  create_at timestamptz [not null,default: `now()`]
}

Table accounts as A {
  id bigser [pk]
  owner varchar [ref:> U.username,not null]
  balance bigint [not null]
  currency varchar  [not null]
  created_at timestamp [not null,default: `now()`] 

  Indexes {
    (owner, currency) [unique]
  }
}

Table entries {
  id bigint [pk]
  account_id bigint [ref : > A.id,not null] 
  
  amount bigint [not null]
  created_at timestamp [not null,default: `now()`]
  
Indexes {
  account_id
}
}

Table transfers {
  id bigint [pk]
  from_account_id bigint [ref : > A.id,not null]
  to_account_id bigint [ref : > A.id,not null]
  amount  bigint [not null]
  created_at timestamp [not null,default: `now()`]

  
Indexes {
  from_account_id
  to_account_id
  (from_account_id,to_account_id)
}
}
~~~

新建数据库迁移：

~~~shell
migrate create -ext sql -dir db/migration -seq add_users
~~~

出现了错误

~~~sql
make migrateup
migrate -path simplebank/db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" -verbose up
2024/10/20 15:11:16 Start buffering 2/u add_users
2024/10/20 15:11:16 Read and execute 2/u add_users
2024/10/20 15:11:16 error: migration failed: syntax error at or near "00" (column 69) in line 6: CREATE TABLE "user" (
  "username" carchar PRIMARY KEY,
  "hashed_paassword" varchar NOT NULL,
  "full_name" varchar NOT NULL,
  "email" varchar UNIQUE NOT NULL,
  "password_changed_at" timestamp NOT NULL DEFAULT (0001-01-01 00:00:00Z),
  "create_at" timestamptz NOT NULL DEFAULT (now())
);


ALTER TABLE "accounts" ADD FOREIGN KEY ("owner") REFERENCES "user" ("username");

--CREATE UNIQUE INDEX ON "accounts" ("owner", "currency");
ALTER TABLE "acounts" ADD CONSTRAINT "owner_currency-unique" UNIQUE ("owner", "currency") (details: pq: syntax error at or near "00")
make: *** [migrateup] 错误 1
~~~

原因：违反了外键约束

### 十七.迁移失败原因

**sql语句写错了 IF写成ID**

**理解去除外键等 **

#### 2.问题：

在执行数据库迁移时，出现的错误是因为在 `accounts` 表上有外键依赖 (`transfers` 表中的 `transfers_from_account_id_fkey` 和 `transfers_to_account_id_fkey` 约束依赖于 `accounts` 表)。当你尝试删除 `accounts` 表时，PostgreSQL 不允许删除这个表，因为还有其他表（如 `transfers`）依赖它。

但是执行了migrateup指令就会出现脏读现象 使得数据库版本变为2 所以我们要先回退到1版本

~~~sql
make migratedown
migrate -path simplebank/db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" -verbose down
2024/10/20 15:15:34 Are you sure you want to apply all down migrations? [y/N]
y
2024/10/20 15:15:36 Applying all down migrations
2024/10/20 15:15:36 error: Dirty database version 2. Fix and force version.
make: *** [migratedown] 错误 1
~~~

修改迁移表的值为 FALSE：没管用

~~~sql
执行migratedown操作时失败，并出现错误信息 “cannot drop table accounts because other objects depend on it”，这表明accounts表有其他数据库对象依赖于它。
原因包括：
transfers表中的外键约束引用了accounts表。
直接删除含外键的表会引发错误。
建议：
修改迁移脚本，先删除依赖的对象，如约束、触发器、视图等。
使用CASCADE选项强制删除所有依赖的对象。
在 makefile 中为migrate命令添加条件检查。
可能的迁移修正示例：
DROP TABLE IF EXISTS transfers CASCADE;
DROP TABLE IF EXISTS accounts;
~~~

#### 3.解除外键约束

~~~sql
解决方案：
你可以按以下步骤修改你的迁移文件，确保先删除外键约束，再删除相关的表。

删除外键约束： 在迁移文件中，先删除 transfers 表中的外键约束：
ALTER TABLE transfers DROP CONSTRAINT IF EXISTS transfers_from_account_id_fkey;
ALTER TABLE transfers DROP CONSTRAINT IF EXISTS transfers_to_account_id_fkey;

删除表： 然后，按顺序删除表：
DROP TABLE IF EXISTS transfers;
DROP TABLE IF EXISTS entries;
DROP TABLE IF EXISTS accounts;
~~~

migrate 出现的错误可能就是   把外键依赖删除然后在执行数据库迁移语句

---

有没有可能你在写数据库迁移的时候就没有在migratedown中加入 删除外键约束的语句呢从而导致这么麻烦？？？

---



编写migratedown时操作是和migrateup相反的

~~~

~~~



#### 4.问题：

~~~sql
make migratedown1
migrate -path simplebank/db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" -verbose down 1
2024/10/20 17:22:21 error: Dirty database version 1. Fix and force version.
make: *** [migratedown1] 错误 1
~~~

这个错误信息表明数据库处于 "dirty" 状态，通常意味着上一次的迁移未成功完成，导致数据库的迁移版本与实际执行情况不一致。要解决这个问题，你可以尝试以下步骤：

#### 5.解决：

检查数据库的迁移状态

使用 `migrate` 查看当前的迁移状态，以确认哪个版本是 dirty：

```sql
migrate -path simplebank/db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" version
```

这将显示当前数据库的版本号以及 dirty 状态。

强制迁移版本

使用 `migrate force` 命令将数据库状态恢复为干净的版本。你可以将其设置为版本 1，这样可以清除 "dirty" 标志，同时保留当前的版本号。

执行以下命令：

```sql
migrate -path simplebank/db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" force 1
```

这个命令不会运行任何迁移，它只是将数据库的迁移版本重置为 1，同时将脏状态清除。



####  6.检查当前数据库中的约束名

如果不确定数据库中的约束名，可以通过以下 SQL 查询当前表中的约束名称：

```sql
SELECT conname FROM pg_constraint WHERE conrelid = 'accounts'::regclass;
```

每条语句后面要写分号啊啊啊啊！！！！！

### 十八. user_test.go

~~~go
package db

import (
	"context"
	util "project/simplebank/util"
	"testing"
	"time"

	"github.com/stretchr/testify/require"
)

func createRandomUser(t *testing.T) User {

	arg := CreateUserParams{
		Username:       util.RandomOwner(),
		HashedPassword: "secret",
		FullName:       util.RandomOwner(),
		Email:          util.RandomEmail(),
	}

	user, err := testStore.CreateUser(context.Background(), arg)
	require.NoError(t, err)
	require.NotEmpty(t, user)

	require.Equal(t, arg.Username, user.Username)
	require.Equal(t, arg.HashedPassword, user.HashedPassword)
	require.Equal(t, arg.FullName, user.FullName)
	require.Equal(t, arg.Email, user.Email)

	require.NotZero(t, user.CreateAt)

	return user
}

func TestCreateUser(t *testing.T) {
	createRandomUser(t)
}

func TestGetUser(t *testing.T) {
	user1 := createRandomUser(t)
	user2, err := testStore.GetUser(context.Background(), user1.Username)
	require.NoError(t, err)
	require.NotEmpty(t, user2)

	require.Equal(t, user1.Username, user2.Username)
	require.Equal(t, user1.HashedPassword, user2.HashedPassword)
	require.Equal(t, user1.FullName, user2.FullName)
	require.Equal(t, user1.Email, user2.Email)
	require.WithinDuration(t, user1.PasswordChangedAt.Time, user2.PasswordChangedAt.Time, time.Second)
	require.WithinDuration(t, user1.CreateAt.Time, user2.CreateAt.Time, time.Second)
}

~~~

**在第29行代码有一个断言语句判断 ：**

~~~go
require.True(t, user.PasswordChangedAt.Time.IsZero())
~~~

这个语句目前不能通过测试 往后看吧看看是么时候找到问题



#### 1. 10.23外键约束问题

运行真个包测试出现的问题

~~~shell
这个外键错误提示 "ERROR: insert or update on table"accounts"violates foreign key constraint"accounts_owner_fkey"(SQLSTATE 23503)" 意味着在尝试往 "accounts" 表中插入或更新数据时违反了名为 "accounts_owner_fkey" 的外键约束。
~~~



**应该是    一个用户链接到账户 这就是主表与副表的关系  设置外键 将两个表链接到一起**



#### 2.数据库表出现错误

数据库语句就写错了 正常每个表的 id序列都应该是自增的 如果不是这样将会出现以下错误

~~~shell
ERROR: null value in column "id" violates not-null constraint (SQLSTATE 23502)
~~~

我们要重新修改数据库迁移语句

~~~go
CREATE TABLE "accounts" (
  "id" bigserial PRIMARY KEY,
  "owner" varchar NOT NULL,
  "balance" bigint NOT NULL,
  "currency" varchar NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE TABLE "entries" (
  "id" bigserial PRIMARY KEY,
  "account_id" bigint NOT NULL,
  "amount" bigint NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE TABLE "transfers" (
  "id" bigserial PRIMARY KEY,
  "from_account_id" bigint NOT NULL,
  "to_account_id" bigint NOT NULL,
  "amount" bigint NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

ALTER TABLE "entries" ADD FOREIGN KEY ("account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("from_account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("to_account_id") REFERENCES "accounts" ("id");

CREATE INDEX ON "accounts" ("owner");

CREATE INDEX ON "entries" ("account_id");

CREATE INDEX ON "transfers" ("from_account_id");

CREATE INDEX ON "transfers" ("to_account_id");

CREATE INDEX ON "transfers" ("from_account_id", "to_account_id");

COMMENT ON COLUMN "entries"."amount" IS 'can be negative or positive';

COMMENT ON COLUMN "transfers"."amount" IS 'must be positive';

~~~

修改过后 正常运行account_test.go

#### 3.修改状态码

~~~go
	account, err := server.store.CreateAccount(ctx, arg)
	if err != nil {
		if pqErr, ok := err.(*pq.Error); ok {
			switch pqErr.Code.Name() {
			case "foreign_key_violation", "unique_violation":
				ctx.JSON(http.StatusForbidden, errorResponse(err))
				return
			default:
				log.Println(pqErr.Code.Name())
			}
			ctx.JSON(http.StatusInternalServerError, errorResponse(err))
			return
		}

		ctx.JSON(http.StatusOK, account)
	}
~~~

出现错误了 等待明天修改

**10.24**

将上述代码语句修改为

~~~go
if err != nil {
			errCode := db.ErrorCode(err)
			if errCode == db.ForeignKeyViolation || errCode == db.UniqueViolation {
				ctx.JSON(http.StatusForbidden, errorResponse(err))
				return
			}
~~~

成功解决了问题 。 这是为什么呢？？

应该是:

~~~go
if errCode == db.ForeignKeyViolation || errCode == db.UniqueViolation
~~~

这段代码起到了主要i作用

在error.go中

~~~
const (
	ForeignKeyViolation = "23503"
	UniqueViolation     = "23505"
)
~~~

这代表了：

~~~
ForeignKeyViolation 常量的值是 "23503"，它代表 PostgreSQL 中的一个错误代码。当执行的数据库操作违反外键约束时，会触发这个错误。外键约束保证了不同表之间的关系，如果尝试插入、更新或删除的数据并不能被其他表中的相关记录引用，就会抛出这个错误。

UniqueViolation 常量的值是 "23505"，这也是一个 PostgreSQL 错误代码。当向需要唯一值的字段插入了重复的值时，会引发这个错误。违反唯一性约束意味着这样的操作将导致两个记录含有相同的值，这在数据库规则中通常是不允许的，因为唯一约束保护了记录唯一识别数据的能力。
~~~



### 十九.在数据库中安全的存储密码

#### 1.password.go

~~~go
package util

import (
	"fmt"

	"golang.org/x/crypto/bcrypt"
)

func HashPassword(password string) (string, error) {
	hashedPassword, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
	if err != nil {
		return "", fmt.Errorf("哈希加密失败:%w", err)
	}
	return string(hashedPassword), nil

}

// checkPassword
func CheckPassword(password string, hashedPassword string) error {
	return bcrypt.CompareHashAndPassword([]byte(hashedPassword), []byte(password))

}

~~~



#### 2.password_test.go

~~~go
package util

import (
	"testing"

	"github.com/stretchr/testify/require"
	"golang.org/x/crypto/bcrypt"
)

func TestPassword(t *testing.T) {
	password := RandomString(6)

	hashPassword, err := HashPassword(password)
	require.NoError(t, err)
	err = CheckPassword(password, hashPassword)
	require.NoError(t, err)

	wrongPassword := RandomString(6)
	err = CheckPassword(wrongPassword, hashPassword)
	require.EqualError(t, err, bcrypt.ErrMismatchedHashAndPassword.Error())

}

~~~



#### 3.user.go

~~~go
package api

import (
	"net/http"

	db "project/simplebank/db/sqlc"
	util "project/simplebank/util"

	"github.com/gin-gonic/gin"
)

type CreateUserRequest struct {
	Username string `json:"username" binding:"required,alphanum"`
	FullName string `json:"fullname" binding:"required"`
	Email    string `json:"email" binding:"required,email"`
	Password string `json:"password" binding:"required,min=6"`
}

func (server *Server) createUser(ctx *gin.Context) {
	var req CreateUserRequest
	if err := ctx.ShouldBindJSON(&req); err != nil {
		ctx.JSON(http.StatusBadRequest, errorResponse(err))
		return
	}

	hashedPassword, err := util.HashedPassword(req.Password)
	if err != nil {
		ctx.JSON(http.StatusInternalServerError, errorResponse(err))
		return

	}
	arg := db.CreateUserParams{
		Username:       req.Username,
		FullName:       req.FullName,
		Email:          req.Email,
		HashedPassword: hashedPassword,
	}
	account, err := server.store.CreateUser(ctx, arg)
	if err != nil {

		errCode := db.ErrorCode(err)
		//此处只保留一个外键约束
		if errCode == db.UniqueViolation {
			ctx.JSON(http.StatusForbidden, errorResponse(err))
			return
		}
		ctx.JSON(http.StatusInternalServerError, errorResponse(err))
		return
	}
	ctx.JSON(http.StatusOK, account)
}

~~~

返回结果

~~~json
{
    "username": "ZhongHe",
    "hashed_password": "$2a$10$RRGhHuYmPf9tRVPDckNI5.q6VJ1TzG9aFJ12edZglg7kp97vGwtKO",
    "full_name": "ZhongHe Zhao",
    "email": "zhaozhonghe40@gmail.com",
    "password_changed_at": "2024-10-24T07:14:46.169687Z",
    "create_at": "2024-10-24T07:14:46.169687Z"
}
~~~



想让返回结果没有 这个字段

~~~json
 "hashed_password": "$2a$10$RRGhHuYmPf9tRVPDckNI5.q6VJ1TzG9aFJ12edZglg7kp97vGwtKO",
~~~

添加

~~~go
type CreateUserResponse struct {
	Username          string    `json:"username"`
	FullName          string    `json:"full_name"`
	Email             string    `json:"email"`
	PasswordChangedAt time.Time `json:"password_changed_at"`
	CreateAt          time.Time `json:"create_at"`
}

~~~





~~~go
rsp := CreateUserResponse{
		Username:          user.Username,
		FullName:          user.FullName,
		Email:             user.Email,
		PasswordChangedAt: user.PasswordChangedAt.Time,
		CreateAt:          user.CreateAt.Time,
	}

	ctx.JSON(http.StatusOK, rsp)
~~~



### 二十.user_test.go

~~~go
package api

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"net/http/httptest"
	"reflect"
	"testing"

	mockdb "project/simplebank/db/mock"
	db "project/simplebank/db/sqlc"
	"project/simplebank/util"

	"github.com/gin-gonic/gin"
	"github.com/golang/mock/gomock"
	"github.com/stretchr/testify/require"
)

type eqCreateUserParamsMatcher struct {
	arg      db.CreateUserParams
	password string
}

func (e eqCreateUserParamsMatcher) Matches(x interface{}) bool {
	arg, ok := x.(db.CreateUserParams)
	if !ok {
		return false
	}

	err := util.CheckPassword(e.password, arg.HashedPassword)
	if err != nil {
		return false
	}

	e.arg.HashedPassword = arg.HashedPassword
	return reflect.DeepEqual(e.arg, arg)
}

func (e eqCreateUserParamsMatcher) String() string {
	return fmt.Sprintf("matches arg %v and password %v", e.arg, e.password)
}

func EqCreateUserParams(arg db.CreateUserParams, password string) gomock.Matcher {
	return eqCreateUserParamsMatcher{arg, password}
}

func TestCreateUserAPI(t *testing.T) {
	user, password := randomUser(t)

	testCases := []struct {
		name          string
		body          gin.H
		buildStubs    func(store *mockdb.MockStore)
		checkResponse func(recoder *httptest.ResponseRecorder)
	}{
		{
			name: "OK",
			body: gin.H{
				"username":  user.Username,
				"password":  password,
				"full_name": user.FullName,
				"email":     user.Email,
			},
			buildStubs: func(store *mockdb.MockStore) {
				arg := db.CreateUserParams{
					Username:       user.Username,
					FullName:       user.FullName,
					Email:          user.Email,
					HashedPassword: user.HashedPassword,
				}
				store.EXPECT().
					CreateUser(gomock.Any(), EqCreateUserParams(arg, password)).
					Times(1).
					Return(user, nil)
			},
			checkResponse: func(recorder *httptest.ResponseRecorder) {
				fmt.Printf("Response code: %d\n", recorder.Code)
				require.Equal(t, http.StatusOK, recorder.Code)
				requireBodyMatchUser(t, recorder.Body, user)
			},
		},
	}

	for i := range testCases {
		tc := testCases[i]

		t.Run(tc.name, func(t *testing.T) {
			ctrl := gomock.NewController(t)
			defer ctrl.Finish()

			store := mockdb.NewMockStore(ctrl)
			tc.buildStubs(store)

			server := newTestServer(t, store)
			recorder := httptest.NewRecorder()

			// Marshal body data to JSON
			data, err := json.Marshal(tc.body)
			require.NoError(t, err)
			fmt.Printf("Request body: %s\n", string(data)) // 打印请求体
			url := "/users"
			request, err := http.NewRequest(http.MethodPost, url, bytes.NewReader(data))
			require.NoError(t, err)

			server.router.ServeHTTP(recorder, request)
			tc.checkResponse(recorder)
			fmt.Printf("Request body: %v\n", tc.body)
		})
	}
}

func randomUser(t *testing.T) (user db.User, password string) {
	password = util.RandomString(6)
	hashedPassword, err := util.HashedPassword(password)
	require.NoError(t, err)

	user = db.User{
		Username:       util.RandomOwner(),
		HashedPassword: hashedPassword,
		FullName:       util.RandomOwner(),
		Email:          util.RandomEmail(),
	}
	return
}

func requireBodyMatchUser(t *testing.T, body *bytes.Buffer, user db.User) {
	data, err := io.ReadAll(body)
	require.NoError(t, err)

	var gotUser db.User
	err = json.Unmarshal(data, &gotUser)

	require.NoError(t, err)
	require.Equal(t, user.Username, gotUser.Username)
	require.Equal(t, user.FullName, gotUser.FullName)
	require.Equal(t, user.Email, gotUser.Email)
	require.Empty(t, gotUser.HashedPassword)
}
~~~







gomock.Any()这个验证的 准确度太低 任何测试基本都能通过

解决方法 使用 新的自定义匹配器

~~~go
 
type eqCreateUserParamsMatcher struct {
	arg      db.CreateUserParams
	password string
}

func (e eqCreateUserParamsMatcher) Matches(x interface{}) bool {
	arg, ok := x.(db.CreateUserParams)
	if !ok {
		return false
	}

	err := util.CheckPassword(e.password, arg.HashedPassword)
	if err != nil {
		return false
	}

	e.arg.HashedPassword = arg.HashedPassword
	return reflect.DeepEqual(e.arg, arg)
}

func (e eqCreateUserParamsMatcher) String() string {
	return fmt.Sprintf("matches arg %v and password %v", e.arg, e.password)
}

func EqCreateUserParams(arg db.CreateUserParams, password string) gomock.Matcher {
	return eqCreateUserParamsMatcher{arg, password}
}

~~~







#### 1.问题

**长记性 json的字段名错误 我测试了一下午**

~~~go
type CreateUserRequest struct {
	Username string `json:"username" binding:"required,alphanum"`
	FullName string `json:"full_name" binding:"required"`
	Email    string `json:"email" binding:"required,email"`
	Password string `json:"password" binding:"required,min=6"`
}
~~~

FullName string `json:"full_name" binding:"required"`这里的json标签我把full_name 写成了fullname



### 二十一.JWT

#### 1.JSON Web令牌

密钥算法

服务器一般使用RSA 和 RS256来验证令牌

对称算法

非对称算法

必须在服务器代码中 检查令牌的算法标头



JWT令牌的很多问题：

![image-20241026184159996](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241026184159996.png)



RASETO作为替代JWT的安全方案

![image-20241026184634250](C:\Users\30413\AppData\Roaming\Typora\typora-user-images\image-20241026184634250.png)



#### 2.基于令牌的身份验证的工作原理是什么？

基于令牌的身份验证从用户登录至系统、设备或应用程序开始，通常使用密码或安全问题。授权服务器验证初始身份验证，然后发放访问令牌，访问令牌是一小段数据，允许客户端应用程序向 API 服务器发出安全调用或信号。

基于令牌的身份验证的工作原理是为服务器提供第二种高度可靠的方式来验证用户的身份和请求的真实性。

完成该基于令牌的初始身份验证协议后，令牌就像盖了章的票据一样：用户可以在令牌生命周期内连续无缝访问相关资源，而无需重新进行身份验证。 该生命周期在用户注销或退出应用程序时结束，也可由设定的超时协议触发。



#### 3.基于令牌的身份验证有何益处？

基于令牌的身份验证能为多个利益相关者提供许多便利：

- **即时的用户体验**：用户无需在每次返回系统、应用程序或网页时重新输入凭据并重新进行身份验证，只要令牌仍然有效（通常会持续到会话因注销或退出而结束），用户就可以保持即时访问。
- **增加了数字安全性**：基于令牌的身份验证在传统的基于密码或基于服务器的身份验证之上又增加了一道安全保护。通常，令牌比密码更难被窃取、被黑客入侵或以其他方式泄露。
- **管理员控制**：基于令牌的身份验证为管理员提供了对每个用户操作和事项的更精细的控制和可见性。
- **减轻技术负担**：由于令牌生成可以与令牌验证完全分离，因此验证可以由辅助服务（如 Entrust 身份和访问管理解决方案提供的服务）来处理。这将显著减少内部服务器和设备上的负载。



### 二十二.编写令牌



#### make.go

~~~go
package token

import (
	"time"
)

// Maker is an interface for managing tokens
type Maker interface {
	// CreateToken creates a new token for a specific username and duration
	CreateToken(username string, role string, duration time.Duration) (string, *Payload, error)

	// VerifyToken checks if the token is valid or not
	VerifyToken(token string) (*Payload, error)
}

~~~



#### payload.go

~~~~go
package token

import (
	"errors"
	"time"

	"github.com/google/uuid"
)

// Different types of error returned by the VerifyToken function
var (
	ErrInvalidToken = errors.New("token is invalid")
	ErrExpiredToken = errors.New("token has expired")
)

// Payload contains the payload data of the token
type Payload struct {
	ID        uuid.UUID `json:"id"`
	Username  string    `json:"username"`
	Role      string    `json:"role"`
	IssuedAt  time.Time `json:"issued_at"`
	ExpiredAt time.Time `json:"expired_at"`
}

// NewPayload creates a new token payload with a specific username and duration
func NewPayload(username string, role string, duration time.Duration) (*Payload, error) {
	tokenID, err := uuid.NewRandom()
	if err != nil {
		return nil, err
	}

	payload := &Payload{
		ID:        tokenID,
		Username:  username,
		Role:      role,
		IssuedAt:  time.Now(),
		ExpiredAt: time.Now().Add(duration),
	}
	return payload, nil
}

// Valid checks if the token payload is valid or not
func (payload *Payload) Valid() error {
	if time.Now().After(payload.ExpiredAt) {
		return ErrExpiredToken
	}
	return nil
}

~~~~



#### jwt_maker.go

~~~go
package token

import (
	"errors"
	"fmt"
	"time"

	"github.com/dgrijalva/jwt-go"
)

const minSecretKeySize = 32

// JWTMaker is a JSON Web Token maker
type JWTMaker struct {
	secretKey string
}

// NewJWTMaker creates a new JWTMaker
func NewJWTMaker(secretKey string) (Maker, error) {
	if len(secretKey) < minSecretKeySize {
		return nil, fmt.Errorf("invalid key size: must be at least %d characters", minSecretKeySize)
	}
	return &JWTMaker{secretKey}, nil
}

// CreateToken creates a new token for a specific username and duration
func (maker *JWTMaker) CreateToken(username string, role string, duration time.Duration) (string, *Payload, error) {
	payload, err := NewPayload(username, role, duration)
	if err != nil {
		return "", payload, err
	}

	jwtToken := jwt.NewWithClaims(jwt.SigningMethodHS256, payload)
	token, err := jwtToken.SignedString([]byte(maker.secretKey))
	return token, payload, err
}

// VerifyToken checks if the token is valid or not
func (maker *JWTMaker) VerifyToken(token string) (*Payload, error) {
	keyFunc := func(token *jwt.Token) (interface{}, error) {
		_, ok := token.Method.(*jwt.SigningMethodHMAC)
		if !ok {
			return nil, ErrInvalidToken
		}
		return []byte(maker.secretKey), nil
	}

	jwtToken, err := jwt.ParseWithClaims(token, &Payload{}, keyFunc)
	if err != nil {
		verr, ok := err.(*jwt.ValidationError)
		if ok && errors.Is(verr.Inner, ErrExpiredToken) {
			return nil, ErrExpiredToken
		}
		return nil, ErrInvalidToken
	}

	payload, ok := jwtToken.Claims.(*Payload)
	if !ok {
		return nil, ErrInvalidToken
	}

	return payload, nil
}

~~~



jwt_test.go

~~~go
package token

import (
	"testing"
	"time"

	"project/simplebank/util"

	"github.com/dgrijalva/jwt-go"
	"github.com/stretchr/testify/require"
)

func TestJWTMaker(t *testing.T) {
	maker, err := NewJWTMaker(util.RandomString(32))
	require.NoError(t, err)

	username := util.RandomOwner()
	role := util.DepositorRole
	duration := time.Minute

	issuedAt := time.Now()
	expiredAt := issuedAt.Add(duration)

	token, err := maker.CreateToken(username, duration)
	require.NoError(t, err)
	require.NotEmpty(t, token)

	payload, err := maker.VerifyToken(token)
	require.NoError(t, err)
	require.NotEmpty(t, payload)

	require.NotZero(t, payload.ID)
	require.Equal(t, username, payload.Username)
	require.Equal(t, role, payload.Role)
	require.WithinDuration(t, issuedAt, payload.IssuedAt, time.Second)
	require.WithinDuration(t, expiredAt, payload.ExpiredAt, time.Second)
}

func TestExpiredJWTToken(t *testing.T) {
	maker, err := NewJWTMaker(util.RandomString(32))
	require.NoError(t, err)

	token, err := maker.CreateToken(util.RandomOwner(), -time.Minute)
	require.NoError(t, err)
	require.NotEmpty(t, token)

	payload, err := maker.VerifyToken(token)
	require.Error(t, err)
	require.EqualError(t, err, ErrExpiredToken.Error())
	require.Nil(t, payload)
}

func TestInvalidJWTTokenAlgNone(t *testing.T) {
	payload, err := NewPayload(util.RandomOwner(), time.Minute)
	require.NoError(t, err)

	jwtToken := jwt.NewWithClaims(jwt.SigningMethodNone, payload)
	token, err := jwtToken.SignedString(jwt.UnsafeAllowNoneSignatureType)
	require.NoError(t, err)

	maker, err := NewJWTMaker(util.RandomString(32))
	require.NoError(t, err)

	payload, err = maker.VerifyToken(token)
	require.Error(t, err)
	require.EqualError(t, err, ErrInvalidToken.Error())
	require.Nil(t, payload)
}

~~~

作者说 passeto是比JWT更简洁更好用



#### passeto_maker.go

~~~go
package token

import (
	"fmt"
	"time"

	"github.com/aead/chacha20poly1305"
	"github.com/o1egl/paseto"
)

// PasetoMaker is a PASETO token maker
type PasetoMaker struct {
	paseto       *paseto.V2
	symmetricKey []byte
}

// NewPasetoMaker creates a new PasetoMaker
func NewPasetoMaker(symmetricKey string) (Maker, error) {
	if len(symmetricKey) != chacha20poly1305.KeySize {
		return nil, fmt.Errorf("invalid key size: must be exactly %d characters", chacha20poly1305.KeySize)
	}

	maker := &PasetoMaker{
		paseto:       paseto.NewV2(),
		symmetricKey: []byte(symmetricKey),
	}

	return maker, nil
}

// CreateToken creates a new token for a specific username and duration
func (maker *PasetoMaker) CreateToken(username string, duration time.Duration) (string, error) {
	payload, err := NewPayload(username, duration)
	if err != nil {
		return "", err
	}

	return maker.paseto.Encrypt(maker.symmetricKey, payload, nil)

}

// VerifyToken checks if the token is valid or not
func (maker *PasetoMaker) VerifyToken(token string) (*Payload, error) {
	payload := &Payload{}

	err := maker.paseto.Decrypt(token, maker.symmetricKey, payload, nil)
	if err != nil {
		return nil, ErrInvalidToken
	}

	err = payload.Valid()
	if err != nil {
		return nil, err
	}

	return payload, nil
}

~~~



#### paseto_make_test.go

~~~go
package token

import (
	"testing"
	"time"

	"project/simplebank/util"

	"github.com/stretchr/testify/require"
)

func TestPasetoMaker(t *testing.T) {
	maker, err := NewJWTMaker(util.RandomString(32))
	require.NoError(t, err)

	username := util.RandomOwner()
	duration := time.Minute

	issuedAt := time.Now()
	expiredAt := issuedAt.Add(duration)

	token, err := maker.CreateToken(username, duration)
	require.NoError(t, err)
	require.NotEmpty(t, token)

	payload, err := maker.VerifyToken(token)
	require.NoError(t, err)
	require.NotEmpty(t, payload)

	require.NotZero(t, payload.ID)
	require.Equal(t, username, payload.Username)
	require.WithinDuration(t, issuedAt, payload.IssuedAt, time.Second)
	require.WithinDuration(t, expiredAt, payload.ExpiredAt, time.Second)
}

func TestExpiredPasetoToken(t *testing.T) {
	maker, err := NewPasetoMaker(util.RandomString(32))
	require.NoError(t, err)

	token, err := maker.CreateToken(util.RandomOwner(), -time.Minute)
	require.NoError(t, err)
	require.NotEmpty(t, token)

	payload, err := maker.VerifyToken(token)
	require.Error(t, err)
	require.EqualError(t, err, ErrExpiredToken.Error())
	require.Nil(t, payload)
}

//None算法

~~~





### 10.28学习如何用令牌登录api

#### 1.server.go

~~~go
package api

import (
	"fmt"
	db "project/simplebank/db/sqlc"
	"project/simplebank/token"
	"project/simplebank/util"

	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"github.com/go-playground/validator/v10"
)

type Server struct {
	config     util.Config
	store      db.Store
	router     *gin.Engine
	tokenMaker token.Maker
}

// 自定义验证函数，检查 currency 是否为 "USD"
func validCurrency(fl validator.FieldLevel) bool {
	currency := fl.Field().String()
	return currency == "RMB"
}

// 注册自定义验证器
func (server *Server) setupValidator() {
	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
		v.RegisterValidation("currency", validCurrency)
	}
}

func NewServer(config util.Config, store db.Store) (*Server, error) {
	tokenMaker, err := token.NewPasetoMaker(config.TokenSymmetricKey)
	if err != nil {
		fmt.Printf("Key length in bytes: %d\n", len([]byte(config.TokenSymmetricKey)))
		return nil, fmt.Errorf("cannot create token maker: %w", err)
	}

	server := &Server{
		config:     config,
		store:      store,
		tokenMaker: tokenMaker,
	}

	// 注册自定义验证器
	server.setupValidator()
	server.setupRouter()
	return server, nil
}

func (server *Server) setupRouter() {
	router := gin.Default()

	router.POST("/users/login", server.loginUser)

	router.POST("transfers", server.createTransfer)
	router.POST("/accounts", server.createAccount)
	router.GET("/accounts/:id", server.getAccount)
	router.POST("/users", server.createUser)
	router.GET("/accounts", server.listAccounts)

	server.router = router

}

func errorResponse(err error) gin.H {
	return gin.H{"error": err.Error()}
}

func (server *Server) Start(address string) error {
	return server.router.Run(address)
}

~~~







#### 2.user.go

~~~go
package api

import (
	"errors"
	"fmt"
	"net/http"
	"time"

	db "project/simplebank/db/sqlc"
	util "project/simplebank/util"

	"github.com/gin-gonic/gin"
	//"github.com/jackc/pgtype"
)

type CreateUserRequest struct {
	Username string `json:"username" binding:"required,alphanum"`
	FullName string `json:"full_name" binding:"required"`
	Email    string `json:"email" binding:"required,email"`
	Password string `json:"password" binding:"required,min=6"`
}

type UserResponse struct {
	Username          string    `json:"username"`
	FullName          string    `json:"full_name" binding:"required"`
	Email             string    `json:"email"`
	PasswordChangedAt time.Time `json:"password_changed_at"`
	CreateAt          time.Time `json:"create_at"`
}

func newUserResponse(user db.User) UserResponse {
	return UserResponse{
		Username:          user.Username,
		FullName:          user.FullName,
		Email:             user.Email,
		PasswordChangedAt: user.PasswordChangedAt.Time,
		CreateAt:          user.CreateAt.Time,
	}

}

func (server *Server) createUser(ctx *gin.Context) {
	var req CreateUserRequest
	if err := ctx.ShouldBindJSON(&req); err != nil {
		ctx.JSON(http.StatusBadRequest, errorResponse(err))
		return
	}
	fmt.Printf("Received request: %+v\n", req) // 打印请求体

	hashedPassword, err := util.HashedPassword(req.Password)
	if err != nil {
		ctx.JSON(http.StatusInternalServerError, errorResponse(fmt.Errorf("failed to hash password: %v", err)))
		return

	}
	arg := db.CreateUserParams{
		Username:       req.Username,
		FullName:       req.FullName,
		Email:          req.Email,
		HashedPassword: hashedPassword,
	}
	user, err := server.store.CreateUser(ctx, arg)
	if err != nil {
		fmt.Printf("Error creating user: %v\n", err) // 打印错误
		errCode := db.ErrorCode(err)
		//此处只保留一个外键约束
		if errCode == db.UniqueViolation {
			return
		}
		ctx.JSON(http.StatusForbidden, errorResponse(err))
		return
	}

	rsp := newUserResponse(user)
	ctx.JSON(http.StatusOK, rsp)

}

type loginUserRequest struct {
	Username string `json:"username" binding:"required,alphanum"`
	Password string `json:"password" binding:"required,min=6"`
}

type loginUserResponse struct {
	AccessToken string       `json:"access_token"`
	User        UserResponse `json:"user"`
}

func (server *Server) loginUser(ctx *gin.Context) {
	var req loginUserRequest
	if err := ctx.ShouldBindJSON(&req); err != nil {
		ctx.JSON(http.StatusBadRequest, errorResponse(err))
		return
	}

	user, err := server.store.GetUser(ctx, req.Username)
	if err != nil {
		if errors.Is(err, db.ErrRecordNotFound) {
			ctx.JSON(http.StatusNotFound, errorResponse(err))
			return
		}
		ctx.JSON(http.StatusInternalServerError, errorResponse(err))
		return
	}

	err = util.CheckPassword(req.Password, user.HashedPassword)
	if err != nil {
		ctx.JSON(http.StatusUnauthorized, errorResponse(err))
		return
	}

	accessToken, err := server.tokenMaker.CreateToken(
		user.Username,
		server.config.AccessTokenDuration,
	)
	if err != nil {
		ctx.JSON(http.StatusInternalServerError, errorResponse(err))
		return
	}

	rsp := loginUserResponse{

		AccessToken: accessToken,

		User: newUserResponse(user),
	}
	ctx.JSON(http.StatusOK, rsp)

}
~~~



#### 3.问题1

为什么运行transfer_text.go出现了很多错误：



#### 4.解决1

在学习的时候图方便把作者的代码全部拉了下来  在transfer_test.go中 有很多情况 在transfer中并没有实现 导致无法对应这些情况

正常时作者留给你的任务 让你去课后实现这些功能



重新回顾第13集：

模拟数据库进行测试：

确保模拟数据库实现与真是数据库相同的接口

出问题的两段代码：

~~~go

		 {
		 	name: "UnauthorizedUser",
		 	body: gin.H{
		 		"from_account_id": account1.ID,
		 		"to_account_id":   account2.ID,
		 		"amount":          amount,
				"currency":        util.RandomCurrency(),
		 	},

		 	buildStubs: func(store *mockdb.MockStore) {
				store.EXPECT().GetAccount(gomock.Any(), gomock.Eq(account1.ID)).Times(1).Return(account1, nil)
		 		store.EXPECT().GetAccount(gomock.Any(), gomock.Eq(account2.ID)).Times(0)
				store.EXPECT().TransferTx(gomock.Any(), gomock.Any()).Times(0)
		 	},
			checkResponse: func(recorder *httptest.ResponseRecorder) {
		 		require.Equal(t, http.StatusUnauthorized, recorder.Code)
		 	},
		 },
		 {
		 	name: "NoAuthorization",
		 	body: gin.H{
		 		"from_account_id": account1.ID,
		 		"to_account_id":   account2.ID,
		 		"amount":          amount,
		 		"currency":        util.USD,
		 	},

		 	buildStubs: func(store *mockdb.MockStore) {
		 		store.EXPECT().GetAccount(gomock.Any(), gomock.Any()).Times(0)
				store.EXPECT().TransferTx(gomock.Any(), gomock.Any()).Times(0)
			},
		 	checkResponse: func(recorder *httptest.ResponseRecorder) {
		 		require.Equal(t, http.StatusUnauthorized, recorder.Code)
		 	},
		 },
~~~



#### 5.问题2

为什么得到分组用户出错



#### 6.解决2



~~~go
//为什么得到分页的时候用户为空 错误出现在这里
func (q *Queries) ListAccounts(ctx context.Context, arg ListAccountsParams) ([]Account, error) {
	rows, err := q.db.Query(ctx, listAccounts, arg.Owner, arg.Limit, arg.Offset)
	if err != nil {
		return nil, err
	}
	defer rows.Close()
	items := []Account{}
	for rows.Next() {
		var i Account
		if err := rows.Scan(
			&i.ID,
			&i.Owner,
			&i.Balance,
			&i.Currency,
			&i.CreatedAt,
		); err != nil {
			return nil, err
		}
		items = append(items, i)
	}
	if err := rows.Err(); err != nil {
		return nil, err
	}
	return items, nil
}
~~~

rows, err := q.db.Query(ctx, listAccounts, arg.Owner, arg.Limit, arg.Offset) 这里查询的条件有arg.owner 但是我们在测试的时候并没有设置owner 可以显示尝试把owner去掉



#### 11.6日 二十二.身份验证中间件 授权API请求



使用make sqlc 和 make mock 重新为listAccount增加 Owner字段

搞了半天 app.env配置错了 应该是

ACCESS_TOKEN_DURATION=15m

我写成别的了



### 二十三.部署目前的程序



#### 1.对程序进行docker化

运用git部署

**注意**：永远不要将更改直接推送到主分支

1.创建新分支-》推送分支-》产生以下结果-》复制url-》创建标题-》创建拉取请求-》从而可以看到 Files changed 文件的更改

~~~shell
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
remote: 
remote: Create a pull request for 'ft/docker' on GitHub by visiting:
remote:      https://github.com/Whuichenggong/projects/pull/new/ft/docker
remote:
To github.com:Whuichenggong/projects.git
 * [new branch]      ft/docker -> ft/docker
~~~

重新回看第10集 配置工作流 最近这两天了解到了工作流有了更深的理解

~~~
# This workflow will build a golang project
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-go

name: Go

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:

  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.20'

    - name: Build
      run: go build -v ./...

    - name: Test
      run: go test -v ./...

~~~

go语言的工作流模板

giuthub action 相当于将一些列配置放到了github上的一个服务器上 也就是相当于将东西放进了github的服务器

---


#### 11.10日

还是github action问题 终于把 Install golang-migrate解决了



因为： 在最开始推送项目到github的时候 就是因为把项目结构推送错了 ，导致推送到github上的项目根目录没有go.mod文件这造成了很大的错误 导致一直失败

今天又解决了 install golang-migrate问题 因为sudo mv migrate /usr/bin/migrate   把之前的 名称换成 **migrate**就好用了



问题2：

make migratedown migrate -path /db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" -verbose down 2024/11/10 13:30:17 error: open /db/migration\.: The system cannot find the path specified. make: *** [migratedown] 错误 1

在Makefile中的指令的 路径又弄错了 必须让指令能找到位置所在



**卧槽：成功了 绿了 妈的**

牛逼



#### Dockerfile

官方镜像

Dockerfile

~~~dockerfile
# Build stage
FROM golang:1.16-alpine3.13 
WORKDIR /app
COPY . .
RUN go build -o main main.go

EXPOSE 8080 
CMD [ "/app/main" ]

~~~

` docker build -t simplebank:latest ` 使用这个指令构建镜像

images的大小很大

~~~shell
docker images 
REPOSITORY         TAG          IMAGE ID       CREATED         SIZE
simplebank         latest       48621dad3f4d   5 minutes ago   656MB
~~~

分阶段构建可以减少体积

也就是

~~~dockerfile
# Build stage 构建二进制文件
FROM golang:1.23-alpine3.20 AS build
WORKDIR /app
COPY . .
RUN go build -o main main.go

# Production stage 生产环境
FROM alpine:3.20
WORKDIR /app
COPY --from=build /app/main .

EXPOSE 8080 
CMD [ "/app/main" ]


~~~

最终体积

~~~
docker images
REPOSITORY         TAG          IMAGE ID       CREATED         SIZE
simplebank         latest       f64691fae70e   7 seconds ago   27.1MB
~~~



~~~shell
docker ps -a列出容器状态


docker rmi f64691fae70e
Untagged: simplebank:latest
Deleted: sha256:f64691fae70e516b799ed846bbeef10045388dae1932ecafc8b93fb208b403f0


//运行这条指令便启动了容器 监听8080端口
 docker run --name simplebank -p 8080:8080 simplebank:latest
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /users                    --> project/simplebank/api.(*Server).createUser-fm (3 handlers)
[GIN-debug] POST   /users/login              --> project/simplebank/api.(*Server).loginUser-fm (3 handlers)
[GIN-debug] GET    /accounts/:id             --> project/simplebank/api.(*Server).getAccount-fm (4 handlers)
[GIN-debug] POST   /accounts                 --> project/simplebank/api.(*Server).createAccount-fm (4 handlers)
[GIN-debug] GET    /accounts                 --> project/simplebank/api.(*Server).listAccounts-fm (4 handlers)
[GIN-debug] POST   /transfers                --> project/simplebank/api.(*Server).createTransfer-fm (4 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Listening and serving HTTP on 127.0.0.1:1124


//重新启动镜像
PS E:\projects\simplebank> docker rm simplebank
simplebank
PS E:\projects\simplebank> docker run --name simplebank -p 8080:8080 -e GIN_MODE=release simplebank:latest

这样启动就不会有上面的输出了
~~~

` docker container inspect postgres12`  检查网络设置


#### 11.13日

##### 问题：





解决用docker启动后 无法用postman测试接口的问题

~~~ shell
docker run --name simplebank -p 8083:8083 -e GIN_MODE=release -e       DB_SOURCE="postgresql://root:secret@172.17.0.2:5432/simplebank?sslmode=disable" simplebank:latest
~~~



**每次修改完dockerfiles或者什么 要记住重新构建镜像**

~~~SHELL
docker build --no-cache -t simplebank:latest .
~~~

##### 关键：

**先使用调试功能 查看是否正确监听端口**

~~~shell
docker run --name simplebank -p 8080:8080  simplebank:latest
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /users                    --> project/simplebank/api.(*Server).createUser-fm (3 handlers)
[GIN-debug] POST   /users/login              --> project/simplebank/api.(*Server).loginUser-fm (3 handlers)
[GIN-debug] GET    /accounts/:id             --> project/simplebank/api.(*Server).getAccount-fm (4 handlers)
[GIN-debug] POST   /accounts                 --> project/simplebank/api.(*Server).createAccount-fm (4 handlers)
[GIN-debug] GET    /accounts                 --> project/simplebank/api.(*Server).listAccounts-fm (4 handlers)
[GIN-debug] POST   /transfers                --> project/simplebank/api.(*Server).createTransfer-fm (4 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Listening and serving HTTP on 0.0.0.0:8080
~~~



这次在测试的时候 有了反应

~~~shell
PS E:\projects\simplebank> docker run --name simplebank -p 8080:8080 -e GIN_MODE=release -e DB_SOURCE="postgresql://root:secret@172.17.0.2:5432/simplebank?sslmode=disable" simplebank:latest
[GIN] 2024/11/13 - 01:17:23 | 401 |      39.567µs |      172.17.0.1 | GET      "/accounts/1"
~~~



此更改导致了postgres连接出错







#####**不使用ip地址使用用户定义的网络 连接到postrges**

`docker network ls`

~~~do
NETWORK ID     NAME                DRIVER    SCOPE
ca0046b2c82c   bank-network        bridge    local
cf35f34026f7   bridge              bridge    local
1500c05159ef   host                host      local
074a556122c6   none                null      local
fafb76e1721e   start_gvb-network   bridge    local

~~~

桥接网络



##### 查看更详细的网络信息

`docker network inspect bridge`



删除网络:

`docker network rm 0fd871187ef1`

##### 创建自己的网络

``docker network create bank_network`

~~~shell
`0fd871187ef1e3b3bee37ac898e895cf54615e267bd6af9d7b2c045fc5178a14
~~~

##### 连接创建的网络

`docker network connect bank-network`

将postrges12 连接到我们创建的网络

`docker network connect bank-network postgres12`



`docker network inspect bank-network`



###### 得先启动 postrges12



**验证 `postgres12` 容器是否正在运行**： 检查 named 的容器是否正在运行：`postgres12`

```
docker ps -a
```

查找具有名称的容器并检查其状态。如果容器未运行，请启动容器：`postgres12`

```
docker start postgres12
```

**再次将 `postgres12` 连接到网络**： 现在，尝试将容器连接到 ：`postgres12``bank-network`

```
docker network connect bank-network postgres12
```



现在已经成功添加了postrges12

~~~shell
 "ConfigOnly": false,
        "Containers": {
            "7ba14f6dd2f7a81db9264c0814e9686e921b0d86c01b2df325dad4a1cca35c40": {
                "Name": "postgres12",
                "EndpointID": "b3dc1614431f2f11f2b0d6c8bb7f33b529baacefa39521bf522c84a7f526a882",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
~~~

此时查看

` docker container inspect postgres12`

这个容器将会有两段网络



~~~shell
 "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "2ea1e674576863a5e20fe6dda2a3ea265dd11b0223dc4a94bbfa23c57adc66d9",
            "SandboxKey": "/var/run/docker/netns/2ea1e6745768",
            "Ports": {
                "5432/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "5432"
                    }
                ]
            },
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "d85289ea4f7ca088375523781a14955e1b1fc58e5af731fe7f4c48fecba470e6",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bank-network": {
                    "IPAMConfig": {},
                    "Links": null,
                    "Aliases": [
                        "7ba14f6dd2f7"
                    ],
                    "MacAddress": "02:42:ac:12:00:02",
                    "NetworkID": "ca0046b2c82ccb1fe4c996950a815d9c374c58514921c9b919899d8169cb9881",
                    "EndpointID": "b3dc1614431f2f11f2b0d6c8bb7f33b529baacefa39521bf522c84a7f526a882",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DriverOpts": {},
                    "DNSNames": [
                        "postgres12",
                        "7ba14f6dd2f7"
                    ]
                },
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "02:42:ac:11:00:02",
                    "NetworkID": "cf35f34026f787fe91864d7e7a2ab23d482b6a6b956a10d596ae0d9818aa7e16",
                    "EndpointID": "d85289ea4f7ca088375523781a14955e1b1fc58e5af731fe7f4c48fecba470e6",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DriverOpts": null,
                    "DNSNames": null
                }
            }
        }
~~~



重新使用指令



` docker run --name simplebank --network bank-network -p 8080:8080 -e GIN_MODE=release -e DB_SOURCE="postgresql://root:secret@172.17.0.2:5432/simplebank?sslmode=disable" simplebank:latest`

此时 simplebank容器将与postgres12运行在同一个网络上

将172.17.0.2替换成postgres12 因为可以通过名称访问网络



启动容器指令：

~~~
docker run --name simplebank --network bank-network -p 8080:8080 -e GIN_MODE=release -e DB_SOURCE="postgresql://root:secret@postgres12:5432/simplebank?sslmode=disable" simplebank:latest
[GIN] 2024/11/13 - 02:06:27 | 400 |     105.754µs |      172.18.0.1 | POST     "/users/login"
~~~



`docker network inspect bank-network`

~~~
 [
    {
        "Name": "bank-network",
        "Id": "ca0046b2c82ccb1fe4c996950a815d9c374c58514921c9b919899d8169cb9881",
        "Created": "2024-05-10T13:32:42.557489581Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "7ba14f6dd2f7a81db9264c0814e9686e921b0d86c01b2df325dad4a1cca35c40": {
                "Name": "postgres12",
                "EndpointID": "b3dc1614431f2f11f2b0d6c8bb7f33b529baacefa39521bf522c84a7f526a882",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "a76e19ef1c210d1cc4f458ed9b2238db810872417e0a1072e8467dda82663a2a": {
                "Name": "simplebank",
                "EndpointID": "487f3ec81ada3bf84e44af700d0ae930075ce8c683755d789c27cadc7f95ed06",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
~~~

目前有两个容器在自定义的网络中运行

**之后的postrges就可以正常使用了**

更改Makefile文件

~~~Makefile
postgres:
	docker run --name postgres12 --network bank-network -p 5432:5432 -e POSTGRES_USER=root -e POSTGRES_PASSWORD=secret -d postgres:12-alpine
~~~



在github中 pullrequest中可以查看更改 并且 merge 分支到主分支-》然后确认合并—》Delete branch





#### 二十四.docker-compose

https://docs.docker.com



创建docker-compose.yaml文件

~~~
 version: "3.9"
services:
  postgres:
    image: postgres:12-alpine
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=simple_bank
    ports:
      - "5432:5432"
    
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    
    environment:
      - DB_SOURCE=postgresql://root:secret@postgres:5432/simple_bank?sslmode=disable
     
~~~

`docker compose up`

~~~shell
 docker compose up
[+] Running 1/0
 ✔ Container simplebank-api-1  Created            0.0s 
Attaching to api-1, postgres-1
api-1       | [GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.
api-1       |
api-1       | [GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
api-1       |  - using env:     export GIN_MODE=release
api-1       |  - using code:    gin.SetMode(gin.ReleaseMode)
api-1       |
api-1       | [GIN-debug] POST   /users                    --> project/simplebank/api.(*Server).createUser-fm (3 handlers)
api-1       | [GIN-debug] POST   /users/login              --> project/simplebank/api.(*Server).loginUser-fm (3 handlers)
api-1       | [GIN-debug] GET    /accounts/:id             --> project/simplebank/api.(*Server).getAccount-fm (4 handlers)
api-1       | [GIN-debug] POST   /accounts                 --> project/simplebank/api.(*Server).createAccount-fm (4 handlers)
api-1       | [GIN-debug] GET    /accounts                 --> project/simplebank/api.(*Server).listAccounts-fm (4 handlers)
api-1       | [GIN-debug] POST   /transfers                --> project/simplebank/api.(*Server).createTransfer-fm (4 handlers)
api-1       | [GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
api-1       | Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
api-1       | [GIN-debug] Listening and serving HTTP on 0.0.0.0:8080
postgres-1  | The files belonging to this database system will be owned by user "postgres".
postgres-1  | This user must also own the server process.
postgres-1  |
postgres-1  | The database cluster will be initialized with locale "en_US.utf8".
postgres-1  | The default database encoding has accordingly been set to "UTF8".
postgres-1  | The default text search configuration will be set to "english".
postgres-1  |
postgres-1  | Data page checksums are disabled.
postgres-1  |
postgres-1  | fixing permissions on existing directory /var/lib/postgresql/data ... ok
postgres-1  | creating subdirectories ... ok
postgres-1  | selecting dynamic shared memory implementation ... posix
postgres-1  | selecting default max_connections ... 100
postgres-1  | selecting default shared_buffers ... 128MB
postgres-1  | selecting default time zone ... UTC
postgres-1  | creating configuration files ... ok
postgres-1  | running bootstrap script ... ok
postgres-1  | sh: locale: not found
postgres-1  | 2024-11-13 06:50:42.795 UTC [30] WARNING:  no usable system locales were found
postgres-1  | performing post-bootstrap initialization ... ok
postgres-1  | syncing data to disk ... ok
postgres-1  |
postgres-1  |
postgres-1  | Success. You can now start the database server using:
postgres-1  |
postgres-1  |     pg_ctl -D /var/lib/postgresql/data -l logfile start
postgres-1  |
postgres-1  | initdb: warning: enabling "trust" authentication for local connections
postgres-1  | You can change this by editing pg_hba.conf or using the option -A, or
postgres-1  | --auth-local and --auth-host, the next time you run initdb.
postgres-1  | waiting for server to start....2024-11-13 06:50:43.144 UTC [36] LOG:  starting PostgreSQL 12.18 on x86_64-pc-linux-musl, compiled by gcc (Alpine 13.2.1_git20231014) 13.2.1 20231014, 64-bit
postgres-1  | 2024-11-13 06:50:43.146 UTC [36] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres-1  | 2024-11-13 06:50:43.160 UTC [37] LOG:  database system was shut down at 2024-11-13 06:50:43 UTC
postgres-1  | 2024-11-13 06:50:43.164 UTC [36] LOG:  database system is ready to accept connections
postgres-1  |  done
postgres-1  | server started
postgres-1  | CREATE DATABASE
postgres-1  |
postgres-1  |
postgres-1  | /usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*
postgres-1  |
postgres-1  | waiting for server to shut down....2024-11-13 06:50:43.315 UTC [36] LOG:  received fast shutdown request
postgres-1  | 2024-11-13 06:50:43.316 UTC [36] LOG:  aborting any active transactions
postgres-1  | 2024-11-13 06:50:43.318 UTC [36] LOG:  background worker "logical replication launcher" (PID 43) exited with exit code 1
postgres-1  | 2024-11-13 06:50:43.318 UTC [38] LOG:  shutting down
postgres-1  | 2024-11-13 06:50:43.330 UTC [36] LOG:  database system is shut down
postgres-1  |  done
postgres-1  | server stopped
postgres-1  |
postgres-1  | PostgreSQL init process complete; ready for start up.
postgres-1  |
postgres-1  | 2024-11-13 06:50:43.447 UTC [1] LOG:  starting PostgreSQL 12.18 on x86_64-pc-linux-musl, compiled by gcc (Alpine 13.2.1_git20231014) 13.2.1 20231014, 64-bit

postgres-1  | 2024-11-13 06:50:43.447 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
postgres-1  | 2024-11-13 06:50:43.447 UTC [1] LOG:  listening on IPv6 address "::", port 5432
postgres-1  | 2024-11-13 06:50:43.450 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres-1  | 2024-11-13 06:50:43.461 UTC [51] LOG:  database system was shut down at 2024-11-13 06:50:43 UTC
postgres-1  | 2024-11-13 06:50:43.465 UTC [1] LOG:  database system is ready to accept connections
~~~





构建镜像完成后

~~~shell
docker images
REPOSITORY         TAG          IMAGE ID       CREATED        SIZE
simplebank-api     latest       eb772c9e932f   6 hours ago    27.1MB
simplebank         latest       9f145f0ce89f   6 hours ago    27.1MB
~~~



查看占用端口的进程



~~~shell
`netstat -ano | findstr :5432`
  TCP    0.0.0.0:5432           0.0.0.0:0              LISTENING       30352
  TCP    [::]:5432              [::]:0                 LISTENING       30352
  TCP    [::1]:5432             [::]:0                 LISTENING       35464
PS E:\projects\simplebank> `tasklist /FI "PID eq 30352"``

映像名称                       PID 会话名              会话#       内存使用
========================= ======== ================ =========== ============
com.docker.backend.exe       30352 Console                    2    117,104 K
PS E:\projects\simplebank> `tasklist /FI "PID eq 35464"``

映像名称                       PID 会话名              会话#       内存使用
========================= ======== ================ =========== ============
wslrelay.exe                 35464 Console                    2      8,328 K
~~~



在 Windows 上（终止进程）：

```shell
taskkill /PID 30352 /F
taskkill /PID 35464 /F
```



`docker ps`

~~~
CONTAINER ID   IMAGE                COMMAND                   CREATED             STATUS         PORTS                                      NAMES
dab18d564f9c   postgres:12-alpine   "docker-entrypoint.s…"   About an hour ago   Up 7 minutes   0.0.0.0:5432->5432/tcp                     simplebank-postgres-1
c4c37a8a870a   simplebank-api       "/app/main"               About an hour ago   Up 7 minutes   0.0.0.0:8080->8080/tcp                     simplebank-api-1
~~~



`docker network inspect simplebank_default`

两个服务容器实际在同一个网络上运行

~~~
 docker network inspect simplebank_default
[
    {
        "Name": "simplebank_default",
        "Id": "fab69439b1a55525d81fa70d9e789c3b6d51ba8d7899924deb8413fb724ca951",
        "Created": "2024-11-13T05:28:49.358856507Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "c4c37a8a870a75e9fa626c7034dd935c8f3afdb86c5e2c37b012503bff9c7ab7": {
                "Name": "simplebank-api-1",
                "EndpointID": "8a247db55db70983d6b2d619caef09bf2593964daa02be5773448fbd74f9d791",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            },
            "dab18d564f9c4554ef255e50205be2f4dd9c1fada3391dde698d7717d0e642ff": {
                "Name": "simplebank-postgres-1",
                "EndpointID": "2b9220ab1bcc031b29631a2ecb462a48a475a722b10592c478124c03d95e29df",
                "MacAddress": "02:42:ac:14:00:03",
                "IPv4Address": "172.20.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "default",
            "com.docker.compose.project": "simplebank",
            "com.docker.compose.version": "2.24.6"
        }
    }
]
~~~



`docker compose down`

删除现在所有网络



Dockerfile

~~~shell
# Build stage 构建二进制文件
FROM golang:1.23-alpine3.20 AS build
WORKDIR /app
COPY . .
RUN go build -o main main.go

# Run stage 
FROM alpine:3.20
WORKDIR /app
COPY --from=build /app/main .
COPY app.env .
# 这一步可以解决2024/11/13 08:08:06 cannot load config:Config File "app" Not Found in "[/app]"

EXPOSE 8080 
CMD [ "/app/main" ]


~~~

操 最后一刻验证成功了

用终端输入指令 带入参数 172.17.0.2 这样 viper可以自动读取配置

~~~shell
docker run --name simplebank -p 8080:8080 -e GIN_MODE=release -e DB_SOURCE="postgresql://root:secret@172.17.0.2:5432/simple_bank?sslmode=disable" simplebank:latest
Received request: {Username:Zhonghe FullName:zhaohzonghe Email:3041322213@qq.com Password:zzh123456}
[GIN] 2024/11/13 - 12:54:07 | 200 |   57.106456ms |      172.17.0.1 | POST     "/users"

~~~

app.env中的配置

~~~
DATABASE_URL=postgres://root:secret@localhost:5432/simple_bank?sslmode=disable
MIGRATION_URL=project/simplebank/db/migration
HTTPServerAddress=0.0.0.0:8080
TOKEN_SYMMETRIC_KEY=12345678901234567890123456789012
ACCESS_TOKEN_DURATION=15m
~~~


#### 11.18日



`docker ps`

`docker network inspect simplebank_default`

~~~shell
CONTAINER ID   IMAGE                COMMAND                   CREATED              STATUS              PORTS                    NAMES
81aa7c463a58   postgres:12-alpine   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:5432->5432/tcp   simplebank-postgres-1
047f0bb9fbc8   simplebank-api       "/app/main"               About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp   simplebank-api-1
PS E:\projects\simplebank> 
[
    {
        "Name": "simplebank_default",
        "Id": "9afc6c5d5e9252f2161f204008596b067fceecd49ac5a9171910c58f4717e205",
        "Created": "2024-11-18T11:05:59.526768414Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "047f0bb9fbc8fdbe07cc311b134c00ff27cb0a2cbcb4322746a6b30cbbb404bf": {
                "Name": "simplebank-api-1",
                "EndpointID": "b607176500386abe6ac7ad27f31d9c453a3f2087dacade426d50a72b1e30b585",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "81aa7c463a58ad777dd3d99f9ba3c442c024c02d0f91be924903ffa423f99426": {
                "Name": "simplebank-postgres-1",
                "EndpointID": "ce4ec4fda5631c27ba1e8c96503ef86f1bfea3bc8f563ecf7528c75ca91f1bb6",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "default",
            "com.docker.compose.project": "simplebank",
            "com.docker.compose.version": "2.24.6"
        }
    }
]
~~~



两个服务器运行在同一个网络 通过名字彼此发现自己

~~~
 "Containers": {
            "047f0bb9fbc8fdbe07cc311b134c00ff27cb0a2cbcb4322746a6b30cbbb404bf": {
                "Name": "simplebank-api-1",
                "EndpointID": "b607176500386abe6ac7ad27f31d9c453a3f2087dacade426d50a72b1e30b585",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "81aa7c463a58ad777dd3d99f9ba3c442c024c02d0f91be924903ffa423f99426": {
                "Name": "simplebank-postgres-1",
                "EndpointID": "ce4ec4fda5631c27ba1e8c96503ef86f1bfea3bc8f563ecf7528c75ca91f1bb6",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
~~~



目前是链接不上数据库的因为没有执行数据库迁移

重新构建docker-compose docker files文件





`docker compose down`
[+] Running 3/3
✔ Container simplebank-postgres-1  Removed                                                                                                                               0.7s
✔ Container simplebank-api-1       Removed                                                                                                                               0.6s
✔ Network simplebank_default       Removed

删除目前所有容器和网络



使用`docker rmi ….`

删除simplebank_api镜像



##### 出错

ERROR [api internal] load metadata for docker.io/library/builder:latest

错误的核心在于 `tar` 解压的文件名与你 `mv` 命令中期望的文件名不匹配。具体表现为 `tar` 解压生成的文件名并不是 `migrate.linux-amd64`，而是 `migrate`。



##### 解决方法

**1. 修改 `RUN` 命令中的文件名引用**

根据错误日志，`tar` 解压后生成的文件名是 `migrate`，而非 `migrate.linux-amd64`。因此，`mv` 命令应改为直接操作 `migrate`：

```
dockerfile复制代码RUN curl -L https://github.com/golang-migrate/migrate/releases/download/v4.17.0/migrate.linux-amd64.tar.gz \
    | tar -xz && mv migrate /app/migrate
```

这将确保正确地将解压出的 `migrate` 文件移动到 `/app/migrate`。

**2. 验证文件解压和路径**

为了确保过程正确，可以在 `RUN` 指令中加入调试信息以打印文件列表：

```
dockerfile复制代码RUN curl -L https://github.com/golang-migrate/migrate/releases/download/v4.17.0/migrate.linux-amd64.tar.gz \
    | tar -xz && ls -l && mv migrate /app/migrate
```



原因：要保持 builer同意 我写成了一个build  另一个builder 因该换成build

~~~
COPY --from=build /app/main .
COPY --from=build /app/migrate /usr/bin/migrate   
~~~



欧克解决了

完整的 dockerfile

~~~docker
# Build stage
FROM golang:1.23-alpine3.20 AS build
WORKDIR /app
COPY . .
RUN go build -o main main.go
RUN apk add curl
RUN curl -L  https://github.com/golang-migrate/migrate/releases/download/v4.17.0/migrate.linux-amd64.tar.gz | tar xvz && mv migrate /app/migrate
     

# Run stage
FROM alpine:3.20
WORKDIR /app
COPY --from=build /app/main .
COPY --from=build /app/migrate /usr/bin/migrate
COPY app.env .
COPY start.sh . 
COPY db/migration ./migration

EXPOSE 8080 
CMD [ "/app/main" ]
ENTRYPOINT [ "/app/start.sh" ]


~~~





完整的 docker-compose.yaml



~~~docker
version: "3.9"
services:
  postgres:
    image: postgres:12-alpine
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=simple_bank
    ports:
      - "5432:5432"
    
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    
    environment:
      - DB_SOURCE=postgresql://root:secret@postgres:5432/simple_bank?sslmode=disable
    depends_on:
      - postgres
~~~



下载wait-for工具

`mv "C:\Users\30413\Downloads\wait-for" ./wait-for.sh`





目前测试api问题：

~~~
{ 
    "error": "failed to connect to `user=root database=simple_bank`:\n\t127.0.0.1:5432 (localhost): dial error: dial tcp 127.0.0.1:5432: connect: connection refused\n\t[::1]:5432 (localhost): dial error: dial tcp [::1]:5432: connect: cannot assign requested address"
}
~~~





#### 二.11.22日

##### 解决上次的问题

无论怎么样构建无法用postman接口调试



这是因为 你在

star.sh中

~~~
#!/bin/sh

set -e

echo "run db migrations"
/app/migrate -path /app/migration -database "$DB_SOURCE" -verbose up

echo "start the app"
exec "$@"

~~~

**解决**

使用的连接数据库的 参数是 $DB_SOURCE" 但是你在app.env中配置的名字不是DB_SOURCE 是DATABASE_URL 这种错误造成的原因可能是目前你并不了解一些列的工具是如何真正使用的没有真正了解





之前的配置 都是用DATABASE_URL来配置的

~~~
DATABASE_URL=postgres://root:secret@localhost:5432/simple_bank?sslmode=disable
MIGRATION_URL=project/simplebank/db/migration
HTTPServerAddress=0.0.0.0:8080
TOKEN_SYMMETRIC_KEY=12345678901234567890123456789012
ACCESS_TOKEN_DURATION=15m
~~~

更改为 DB_SOURCE后api测试成功



#### 三.11.23日

部署应用程序

##### 创建AWS(最大的云提供商)账户部署应用程序

地址 https://aws.amazon.com/free/

emmm不知道银行卡的cvv



##### 自动构建docker镜像并推送到AWS ECR

1.创建一个存储库存储docker镜像

将docker 镜像推送到CLI
每当新代码合并到主分支时 我们将使用 Github Actions自动构建标记和推送镜像

deploy.yml 关键



**目前没有招商卡无法使用AWS 先使用快过期的aliyun试一试**

添加go到linux环境

~~~
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@iZt4nbaeq7uzlvq978l1xqZ simplebank]# ^C
[root@iZt4nbaeq7uzlvq978l1xqZ simplebank]#    export PATH=$PATH:/usr/local/go/bin
[root@iZt4nbaeq7uzlvq978l1xqZ simplebank]# go run main.go
go: downloading github.com/jackc/pgx/v5 v5.7.1
go: downloading github.com/gin-gonic/gin v1.10.0

~~~





#### 四.11.29日 尝试

在仅剩5个月的服务器中 把这个简单的项目部署到服务器上

配置服务器的docker的yum源 否则下载东西很费劲

设置国内镜像【不设置可能会导致拉取镜像失败】
进入/etc/docker文件夹下，修改daemon.json。如果文件不存在则，创建该文件。

daemon.json文件内容如下



~~~
{
"registry-mirrors" : [
    "https://jkfdsf2u.mirror.aliyuncs.com",
    "https://registry.docker-cn.com"
  ],
  "insecure-registries" : [
    "docker-registry.zjq.com"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "10"
  },
  "data-root": "/data/docker"
}
~~~





拉取docker pull镜像

~~~
docker pull postgres:12-alpine


~~~

。。。。 配置不够cpu直接干到100%    看看有没有 简化的方法



11.30日

还是执着一点  弄了一台2核2gb的服务器 用docker部署

首先是源的配置 安装docker 安装docker-compose

然后是构建项目中出现的问题 反复构建

赋予权限等

权限问题在ubuntu中也是一个很重要的问题  哪个用户使用ubuntu也会导致不同的结果



~~~~
从 ls -ld 命令的输出可以看到，/home/ubuntu/projects/simplebank 目录的所有者和所属组都是 ubuntu，权限也允许当前用户进行访问。这意味着该目录的所有权和权限没有问题。

但根据 Git 提示的错误信息，Git 依然检测到目录的所有权问题，因此需要添加该目录到 安全目录 列表中。

解决方案：
运行以下命令，将该目录添加到 Git 的安全目录列表中：


git config --global --add safe.directory /home/ubuntu/projects/simplebank
~~~~



看到希望了

~~~
root@VM-12-4-ubuntu:/home/ubuntu/projects/simplebank# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@VM-12-4-ubuntu:/home/ubuntu/projects/simplebank# docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS    PORTS                                       NAMES
758e9432d178   e054039bb12c   "/app/start.sh /app/…"   27 minutes ago   Created   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   simplebank
~~~



还需要配置数据库吗？？我有点蒙了



docker run与docker start的区别





![image-20241130213549768](/study_photo2/simplebank1.png)





####  五.2024年 11.30日 21：27分  成了把项目成功部署到了云服务器上太不容易了




![[simplebank2.png]]




![[simplebank3.png]]




![[simplebank4.png]]




![[simplebank5.png]]










~~~
root@VM-12-4-ubuntu:/home/ubuntu/projects/simplebank# docker run --name simplebank -p 80:8080 -e GIN_MODE=release -e DB_SOURCE="postgresql://root:secret@172.17.0.2:5432/simple_bank?sslmode=disable" simplebank:latest
run db migrations
2024/11/30 13:25:56 no change
2024/11/30 13:25:56 Finished after 977.24µs
2024/11/30 13:25:56 Closing source and database
start the app
Received request: {Username:Zhonghe FullName:zhaohzonghe Email:3041322213@qq.com Password:zzh123456}
[GIN] 2024/11/30 - 13:26:07 | 200 |   75.179039ms |  202.97.179.126 | POST     "/users"
~~~


![[simplebank6.png]]



![[simplebank7.png]]






为什么把端口8080:8080改成 80:8080就好用了 啊啊啊啊好兴奋 感谢老哥们



从把项目移动到 ubuntu 配置dockers环境

使用docker build构建项目

然后就是用postman测试

这期间 多次使用的

##### Docker 指令

docker run

docker images

docker ps -a

docekr pull

docker build -t simplebank:latest .

docker network create bank_network

docker network rm 0fd871187ef1

docker rm simplebank

docker rmi

docker network connect bank-network postgres12

docker network ls

docker container inspect postgres12



目前服务器中的 postgres12大体网络模式

~~~
root@VM-12-4-ubuntu:/home/ubuntu/projects/simplebank# docker container inspect postgres12
[
    {
        "Id": "100ff1a5f0bf6e1f0447fff800aaa00ba54edc2cf19826eef512a442c2ec3a47",
        "Created": "2024-11-30T09:24:02.785101065Z",
        "Path": "docker-entrypoint.sh",
        "Args": [
            "postgres"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 315509,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2024-11-30T11:54:14.985494404Z",
            "FinishedAt": "2024-11-30T11:35:40.792853655Z"
        },
        "Image": "sha256:486566ce0ca8f59e321b2b5999de4b50237b2c60bcc3414d8a602fb96cb12c6f",
        "ResolvConfPath": "/data/docker/containers/100ff1a5f0bf6e1f0447fff800aaa00ba54edc2cf19826eef512a442c2ec3a47/resolv.conf",
        "HostnamePath": "/data/docker/containers/100ff1a5f0bf6e1f0447fff800aaa00ba54edc2cf19826eef512a442c2ec3a47/hostname",
        "HostsPath": "/data/docker/containers/100ff1a5f0bf6e1f0447fff800aaa00ba54edc2cf19826eef512a442c2ec3a47/hosts",
        "LogPath": "/data/docker/containers/100ff1a5f0bf6e1f0447fff800aaa00ba54edc2cf19826eef512a442c2ec3a47/100ff1a5f0bf6e1f0447fff800aaa00ba54edc2cf19826eef512a442c2ec3a47-json.log",
        "Name": "/postgres12",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {
                    "max-file": "10",
                    "max-size": "10m"
            
            "Networks": {
                "bank_network": {
                    "IPAMConfig": {},
                    "Links": null,
                    "Aliases": [],
                    "MacAddress": "02:42:ac:12:00:02",
                    "DriverOpts": {},
                    "NetworkID": "c2a3ada685148d5607a5a6fc39e1690e5fbd161f0607df5a3a189f74ced100fa",
                    "EndpointID": "07a57c58250657bf968d33d1f93cea6e9225d0cae314648d1b1c639c3811c9c1",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": [
                        "postgres12",
                        "100ff1a5f0bf"
                    ]
                },
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null,
                    "NetworkID": "83e7fddfe207131e6199fb11fb5daa38bf044b67817fba2de02bd7f1639d4bb8",
                    "EndpointID": "d9449d910f4e7be735031acb301f0e418999b091bb8c75450fecf983eca2aa24",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": null
                }
            }
        }
    }
]

~~~







---



还是看跟着课程走一走吧


![[simplebank8.png]]



![[simplebank9.png]]




AWS的EKS


将工作节点 添加到EKS集群 使用 kubectl 连接到集群



##### 如何创建新的EKS集群并向其中添加工作节点

大多都是用AWS目前没有卡还是先不要弄了

学习一下其他的知识



### 一.进阶后端

master haha



管理用户会话

用PASETO JWT作为基于令牌的身份验证

因为这些是无状态设计 这些令牌不会存储到数据库中 寿命应该很短

他们的过期时间通常为10~15分钟 如果token每次都在这么短时间过期重新输入用户名和密码一定不是一个好的体验





刷新令牌

在服务器上维护有状态的会话

它将存储在数据库中 生命周期长



创建一个新的字段添加到app.env中

REFRESH_TOKEN_DURATION=24h



同时config中添加新字段

RefreshTokenDuration time.Duration `mapstructure:"REFRESH_TOKEN_DURATION"`



使用指令 migrate create -ext sql -dir db/migration -seq  <migration_name>

~~~shell
，用于创建一个新的迁移文件。该指令参数的意义如下：<migration_name>表示迁移文件的名称；-ext sql指定迁移文件的扩展名；-dir db/migration定义了迁移文件的存储路径；-seq代表创建顺序迁移文件，并在文件名前加上序号。这个命令会在指定目录下生成两个文件，一个用于执行迁移（.up.sql），另一个用于回滚迁移（.down.sql），以实现数据库的版本控制和变更管理。
~~~





##### add_sessions.up.sql

~~~
CREATE TABLE "sessions" (
  "id" uuid PRIMARY KEY,
  "username" varchar NOT NULL,
  "refresh_token" varchar NOT NULL,
  "user_agent" varchar NOT NULL,
  "client_ip" varchar NOT NULL,
  "is_blocked" boolean NOT NULL DEFAULT false,
  "expires_at" timestamptz NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

ALTER TABLE "sessions" ADD FOREIGN KEY ("username") REFERENCES "users" ("username");

~~~

"is_blocked" boolean NOT NULL DEFAULT false,  添加bool列来阻止会话 以防止刷新令牌被泄露

"expires_at" timestamptz NOT NULL, 刷新令牌的过期时间



ALTER TABLE "sessions" ADD FOREIGN KEY ("username") REFERENCES "users" ("username"); 外键约束





#### 11.30日



理清楚`sqlc generate` 到底是什么意思



依赖于sqlc.yml文件

~~~yaml
version: "2"
sql:
- schema: "./db/migration"
  queries: "./db/query"
  engine: "postgresql"
  gen:
    go: 
      package: "db"
      out: "./db/sqlc"
      sql_package: "pgx/v5"
      emit_json_tags: true
      emit_interface: true
      emit_empty_slices: true
      overrides:
        - db_type: "timestamptz"
          go_type: "time.Time"
        - db_type: "uuid"
          go_type: "github.com/google/uuid.UUID"
~~~





指定一些列路径 自动生成代码到哪个位置

依赖的是.sql文件自动生成 相关的代码


#### 12.2日

加入更多的响应


![[simplebank10.png]]

#### 1.4日 巩固项目知识

**store.go**

`github.com/jackc/pgx/v5/pgxpool`：这是用于连接和操作 PostgreSQL 数据库的第三方包。`pgxpool` 是 `pgx` 库的连接池实现，用于管理数据库连接。


`connPool` 是一个连接池，用于管理数据库连接。它的类型是 `pgxpool.Pool`，这是 `pgx` 包提供的数据库连接池。


```go
type SQLStore struct {
    connPool *pgxpool.Pool
    *Queries
}
```


#### 事务
**exec_tx.go**

```go
package db

  

import (

    "context"

    "fmt"

)

  

// ExecTx executes a function within a database transaction

func (store *SQLStore) execTx(ctx context.Context, fn func(*Queries) error) error {

    tx, err := store.connPool.Begin(ctx)

    if err != nil {

        return err

    }

  

    q := New(tx)

    err = fn(q)

    if err != nil {

        if rbErr := tx.Rollback(ctx); rbErr != nil {

            return fmt.Errorf("tx err: %v, rb err: %v", err, rbErr)

        }

        return err

    }

  

    return tx.Commit(ctx)

}
```

为项目添加事务！


代码功能：
- 开始一个新的数据库事务。
- 创建一个可以在事务中执行查询的 `Queries` 对象。
- 执行你传入的函数 `fn`，该函数会在事务内执行操作。
- 如果函数执行成功，则提交事务。
- 如果发生错误，则回滚事务，并返回错误信息。
- 
它保证了事务的一致性和原子性：如果在事务执行过程中遇到任何错误，所有已执行的操作都会被回滚，确保数据不处于半完成的状态。


#### 死锁


**tx_transfer.go**

```go
package db

  

import (

    "context"

)

  

// TransferTxParams contains the input parameters of the transfer transaction

type TransferTxParams struct {

    FromAccountID int64 `json:"from_account_id"`

    ToAccountID   int64 `json:"to_account_id"`

    Amount        int64 `json:"amount"`

}

  

// TransferTxResult is the result of the transfer transaction

type TransferTxResult struct {

    Transfer    Transfer `json:"transfer"`

    FromAccount Account  `json:"from_account"`

    ToAccount   Account  `json:"to_account"`

    FromEntry   Entry    `json:"from_entry"`

    ToEntry     Entry    `json:"to_entry"`

}

  

var txKey = struct{}{}

  

// TransferTx performs a money transfer from one account to the other.

// It creates the transfer, add account entries, and update accounts' balance within a database transaction

func (store *SQLStore) TransferTx(ctx context.Context, arg TransferTxParams) (TransferTxResult, error) {

    var result TransferTxResult

  

    err := store.execTx(ctx, func(q *Queries) error {

        var err error

  

        //txName := ctx.Value(txKey)

  

        //fmt.Println(txName, "Create transfer")

        result.Transfer, err = q.CreateTransfer(ctx, CreateTransferParams{

            FromAccountID: arg.FromAccountID,

            ToAccountID:   arg.ToAccountID,

            Amount:        arg.Amount,

        })

        if err != nil {

            return err

        }

  

        //fmt.Println(txName, "Create entry 1")

        result.FromEntry, err = q.CreateEntry(ctx, CreateEntryParams{

            AccountID: arg.FromAccountID,

            Amount:    -arg.Amount,

        })

        if err != nil {

            return err

        }

  

        //  fmt.Println(txName, "Create entry 2")

        result.ToEntry, err = q.CreateEntry(ctx, CreateEntryParams{

            AccountID: arg.ToAccountID,

            Amount:    arg.Amount,

        })

        if err != nil {

            return err

        }

  

        if arg.FromAccountID < arg.ToAccountID {

            result.FromAccount, result.ToAccount, err = addmoney(ctx, q, arg.FromAccountID, -arg.Amount, arg.ToAccountID, arg.Amount)

  

        } else {

            result.ToAccount, result.ToAccount, err = addmoney(ctx, q, arg.ToAccountID, arg.Amount, arg.FromAccountID, -arg.Amount)

        }

        return err

  

    })

  

    return result, err

}

  

func addmoney(

    ctx context.Context,

    q *Queries,

    accountID1 int64,

    amount1 int64,

    accountID2 int64,

    amount2 int64,

  

) (account1 Account, account2 Account, err error) {

    account1, err = q.AddAccountBalance(ctx, AddAccountBalanceParams{

        ID:     accountID1,

        Amount: amount1,

    })

    if err != nil {

        return

    }

    account2, err = q.AddAccountBalance(ctx, AddAccountBalanceParams{

        ID:     accountID2,

        Amount: amount2,

    })

  

    return

}****
```



```go
  if arg.FromAccountID < arg.ToAccountID {

            result.FromAccount, result.ToAccount, err = addmoney(ctx, q, arg.FromAccountID, -arg.Amount, arg.ToAccountID, arg.Amount)

  

        } else {

            result.ToAccount, result.ToAccount, err = addmoney(ctx, q, arg.ToAccountID, arg.Amount, arg.FromAccountID, -arg.Amount)

        }

        return err

  

    })


```

其中这段代码避免了死锁，规定了谁先增加钱，谁先减少钱



### 1.9日回家了 猛猛干！

又发现了一个错误：

model.go中 


pgtype.Timestamp 与 time.time有什么区别

因为类型的错误导致 user_test.go中数据库模拟出错

#### 问题：
为什么多次使用sqlc生成代码 只有 User结构体的 字段没有变成time.time

#### 解决：
而是当初编辑 migration 代码的时候就没有定义好 时间这个字段导致sqlc生成也无法转换

理解overrides: 作用
overrides:

        - db_type: "timestamptz"

          go_type: "time.Time"

        - db_type: "uuid"

          go_type: "github.com/google/uuid.UUID"

意思是 把数据库中 `timestamptz` 这个类型转换为 time.Time类型
但是在当初编写结构的时候 出现了差错 属性编写成了  `timestamp`

将数据库db中属性类型 被生成的go语言代码将会转换到go语言类型

**正确结构：**

`CREATE TABLE "users" (

  "username" varchar PRIMARY KEY,

  "hashed_password" varchar NOT NULL,

  "full_name" varchar NOT NULL,

  "email" varchar UNIQUE NOT NULL,

  "password_changed_at" timestamptz NOT NULL DEFAULT('0001-01-01 00:00:00Z'),  

  "created_at" timestamptz NOT NULL DEFAULT (now())

);
`



`time.Time` (标准库)

**用途**：用于表示日期和时间，支持丰富的时间计算、格式化和解析功能。

`pgtype.Timestamp` (pgx 库)

**用途**：用于在与 PostgreSQL 数据库交互时表示时间戳。它是 `pgx` 用于处理 PostgreSQL 中 `timestamp` 数据类型的特殊类型。



很多api的测试的状态码出现了很多问题：

逻辑没有设置明白，导致gomock测试的时候无法对应正确的状态码


写测试的时候增强测试， gomockEQ

##### 18集 gomock讲解 
自己编辑gomock 加强测试


30

#### 什么是Kubernetes


Worker Node 

![[Kubernets.png]]
![[Kubernetes2.png]]

AmazonEKS



---

使用 dbdocs 和 DBML的CLI工具直接从终端生成DB文档页面以及SQL模式代码

使用工具 dbml


```shell
dbdocs build doc/db.dbml
√ Parsing file content
? Username: (Whuichenggong)
? Project name:  Simple Bank Database
‼ Project 'Simple Bank Database' is public, consider setting password or restricting access to it
√ Done. Visit: https://dbdocs.io/Whuichenggong/Simple-Bank-Database

i Thanks for using dbdocs! We'd love to hear your feedback: https://form.jotform.com/200962053361448
```

使用CLI生成SQL代码

```shell
dbml2sql --postgres -o doc/schema.sql doc/db.dbml
  ✔ Generated SQL dump file (PostgreSQL): schema.sql
```


### 二. gRPC

gRPC 试图解决什么问题（通信！）

协议缓冲区（Protocol buffer） 生成存根

Protocol buffer（protoc）生成服务器和客户端存根代码

HTTP2：


4 type of gRPC

[流式传输和响应是一种技术，服务器可以分段发送响应数据，无需等待所有数据生成完毕。这种方式特别适合于大语言模型的文本生成任务，因为它可以改善用户的等待体验。流式调用的优势包括提升用户体验、减少服务器压力和增强交互性](https://www.bing.com/ck/a?!&&p=088cb7e6fc4bdc23320c6a1f65b1622227b0f4a103cf33247250c54c328feb72JmltdHM9MTczNjg5OTIwMA&ptn=3&ver=2&hsh=4&fclid=07769bec-ea32-6efb-3f13-888aeb716f1f&u=a1aHR0cHM6Ly93d3cueGluZmluaXRlLm5ldC90L3RvcGljLzY1NDk&ntb=1)[1](https://www.bing.com/ck/a?!&&p=088cb7e6fc4bdc23320c6a1f65b1622227b0f4a103cf33247250c54c328feb72JmltdHM9MTczNjg5OTIwMA&ptn=3&ver=2&hsh=4&fclid=07769bec-ea32-6efb-3f13-888aeb716f1f&u=a1aHR0cHM6Ly93d3cueGluZmluaXRlLm5ldC90L3RvcGljLzY1NDk&ntb=1)[2](https://www.bing.com/ck/a?!&&p=ba35387dd4a09d9d1bab2ad8a8d1f0e7b01ccb45adda1b051474fd89001c69ccJmltdHM9MTczNjg5OTIwMA&ptn=3&ver=2&hsh=4&fclid=07769bec-ea32-6efb-3f13-888aeb716f1f&u=a1aHR0cHM6Ly9qdWVqaW4uY24vcG9zdC83MzczODA4MjAyODY4MTI5ODQ0&ntb=1)[3](https://www.bing.com/ck/a?!&&p=ede55f6c7368df594f863c11bbe5ea91c6746b3140d1c26dd6e558b568c0d19bJmltdHM9MTczNjg5OTIwMA&ptn=3&ver=2&hsh=4&fclid=07769bec-ea32-6efb-3f13-888aeb716f1f&u=a1aHR0cHM6Ly9qdWVqaW4uY24vcG9zdC83NDQxMDM4MzU3Nzg5MDMyNDg1&ntb=1)[4](https://www.bing.com/ck/a?!&&p=65d1b0df5c58d0045b0d6a6f9a83e95a66baaee12a1639d01216b542ef84f802JmltdHM9MTczNjg5OTIwMA&ptn=3&ver=2&hsh=4&fclid=07769bec-ea32-6efb-3f13-888aeb716f1f&u=a1aHR0cHM6Ly9weXRob24ubGFuZ2NoYWluLmFjLmNuL2RvY3MvY29uY2VwdHMvc3RyZWFtaW5nLw&ntb=1)。


![[gRPC.png]]
 

 微服务是gRPC的主场

![[work with gRPC.png]]



gRPC 网关： 


使用protobuf 定义gRPC API

![[gRPC工具.png]]

生成与gRPC框架兼容的Go代码


配置protoc

![[setting protoc.png]]


#### 为项目 api编写 protobuf定义：

为创建用户api编写proto

user.proto

```proto
syntax = "proto3";

  

package pb;

  

import "google/protobuf/timestamp.proto";

  

option go_package = "project/simplebank/pb";

  

message User {

    string username = 1;

    string full_name = 2;

    string email = 3;

    google.protobuf.Timestamp password_changed_at = 4;

    google.protobuf.Timestamp created_at = 5;

}
```


rpc_create_user.proto

```proto
syntax = "proto3";

  

package pb;

  

import "user.proto";

  

option go_package = "project/simplebank/pb";

  

message CreateUserRequest {

    string username = 1;

    string full_name = 2;

    string email = 3;

    string password = 4;

}

message CreateUserResponse {
    User user = 1;
}
```

service_simple_bank.proto

```proto
syntax = "proto3";

  

package pb;

  

import "rpc_create_user.proto";

  

option go_package = "project/simplebank/pb";

  

service Simplebank {

    rpc CreateUser (CreateUserRequest) returns (CreateUserResponse) {}

}
```


#### proto的作用


**高效的序列化与反序列化**

- **序列化**：将内存中的数据结构（如对象、结构体）转换成一种适合存储或传输的格式。
- **反序列化**：将存储或传输的格式数据还原为原始的数据结构。
- Proto 使用的是二进制格式，比传统的 JSON 或 XML 格式更高效，占用的内存更小，传输速度更快。

**跨语言通信**

- 在微服务架构中，不同的服务可能使用不同的编程语言，Proto 可以通过 **gRPC** 实现跨语言的服务调用。通过编写统一的 `.proto` 文件，各个服务之间可以通过 gRPC 协议进行高效、安全的通信。

**接口定义和服务通信**

- Proto 用于定义服务的接口，特别是在使用 gRPC 时，`.proto` 文件不仅定义了数据结构，还定义了服务的远程调用接口。
- 例如，在一个电商系统中，`OrderService` 可以通过 Proto 文件定义创建订单的接口，其他服务可以通过 gRPC 调用这个接口。



### 三. 重构HTTP JSON API

使用gRPC api 而不是HTTP请求


在 service_simple_bank_grpc.pb.go

```go
type SimplebankServer interface {

    CreateUser(context.Context, *CreateUserRequest) (*CreateUserResponse, error)

    LoginUser(context.Context, *LoginUserRequest) (*LoginUserResponse, error)

    mustEmbedUnimplementedSimplebankServer()

}
```
服务器只有实现这个接口才可以成为gRPC服务器

客户端通过简单地执行RPC 来调用服务器

```go
func runGrpcServer(config util.Config, store db.Store) {

    server, err := gapi.NewServer(config, store)

    if err != nil {

        log.Fatal("cannot create gRPCserver %v", err)

    }

    grpcServer := grpc.NewServer()

    pb.RegisterSimplebankServer(grpcServer, server)

    reflection.Register(grpcServer)

  

    listener, err := net.Listen("tcp", config.GRPCServerAddress)

    if err != nil {

        log.Fatal("cannot create Listener")

    }

  

    log.Printf("start gRPC server at %s", listener.Addr().String())

    err = grpcServer.Serve(listener)

    if err != nil {

        log.Fatal("cannot start gRPC server")

    }

}
```





https://github.com/ktr0731/evans 工具

gRPC client 允许在交互式控制台构建和发送gRPC请求



`envans --host localhsot --port 9090 -e repl`

![[gRPC_client.png]]



#### 遇见问题

为什么show service 并没有显示出我展示的两个方法
![[问题.png]]


通过群里解答： 用了图中两个命令就出来了！


### 四.为gRPC服务创建api


#### 问题： 

如何将 db.User 转换成pb.User并返回

#### conversion.go

```go
  

func convertUser(user db.User) *pb.User {

    return &pb.User{

        Username:          user.Username,

        FullName:          user.FullName,

        Email:             user.Email,

        PasswordChangedAt: timestamppb.New(user.PasswordChangedAt),

        CreatedAt:         timestamppb.New(user.CreatedAt),

    }

}
```


#### rpc_create_user.go
```go
  

func (server *Server) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.CreateUserResponse, error) {

  

    hashedPassword, err := util.HashedPassword(req.GetPassword())

    if err != nil {

        return nil, status.Errorf(codes.Internal, "failed to hash password: %s", err)

    }

    arg := db.CreateUserParams{

        Username:       req.GetUsername(),

        FullName:       req.GetFullName(),

        Email:          req.GetEmail(),

        HashedPassword: hashedPassword,

    }

    user, err := server.store.CreateUser(ctx, arg)

    if err != nil {

        errCode := db.ErrorCode(err)

        //此处只保留一个外键约束

        if errCode == db.UniqueViolation {

            return nil, status.Errorf(codes.AlreadyExists, "username or email already exists: %s", err)

  

        }

        return nil, status.Errorf(codes.Internal, "failed to create user: %s", err)

  

    }

    rsp := &pb.CreateUserResponse{

        User: convertUser(user),

    }

  

    return rsp, nil

}
```

#### rpc_login_user.go

```go
  

func (server *Server) LoginUser(ctx context.Context, req *pb.LoginUserRequest) (*pb.LoginUserResponse, error) {

    user, err := server.store.GetUser(ctx, req.GetUsername())

    if err != nil {

        if errors.Is(err, db.ErrRecordNotFound) {

            return nil, status.Errorf(codes.NotFound, "user not found")

        }

        return nil, status.Errorf(codes.Internal, "failed to get user: %s", err)

    }

  

    err = util.CheckPassword(req.Password, user.HashedPassword)

    if err != nil {

        return nil, status.Errorf(codes.NotFound, "Incorrect password")

    }

  

    accessToken, accessPayload, err := server.tokenMaker.CreateToken(

        user.Username,

        server.config.AccessTokenDuration,

    )

    if err != nil {

        return nil, status.Errorf(codes.Internal, "failed to create access token: %s", err)

    }

    refreshToken, refreshPayload, err := server.tokenMaker.CreateToken(

        user.Username,

        server.config.RefreshTokenDuration,

    )

    if err != nil {

        return nil, status.Errorf(codes.Internal, "failed to create refresh token: %s")

    }

    session, err := server.store.CreateSession(ctx, db.CreateSessionParams{

        ID:           refreshPayload.ID,

        Username:     user.Username,

        RefreshToken: refreshToken,

        UserAgent:    "",

        ClientIp:     "",

        IsBlocked:    false,

        ExpiresAt:    refreshPayload.ExpiredAt,

    })

    if err != nil {

        return nil, status.Errorf(codes.Internal, "failed to create session")

    }

  

    rsp := &pb.LoginUserResponse{

        User:                   convertUser(user),

        SessionToken:           session.ID.String(),

        AccessToken:            accessToken,

        RefreshToken:           refreshToken,

        AccessTokenExpiresAt:   timestamppb.New(accessPayload.ExpiredAt),

        Refresh_TokenExpiresAt: timestamppb.New(refreshPayload.ExpiredAt),

    }

  

    return rsp, nil

}
```


#### 网关：

网关是一种能够在不同网络 "网络传输协议"或“协议"协议栈"之间进行数据交换 "数据交换")的设备或服务器。

gRPC Gateway 的作用是 **在 HTTP 和 gRPC 之间提供一个桥梁**，让客户端通过 HTTP 请求与 gRPC 服务进行交互。它主要的作用是将 HTTP 请求转发到 gRPC 服务，同时将 gRPC 的响应转换为 HTTP 响应返回给客户端。



![[http两个包区别.png]]

多看文档！
教程可能是老的，但文档一定是最新的
#### main.go

```go
package main

  

import (

    "context"

    "flag"

    "log"

    "net"

    "net/http"

    "project/simplebank/gapi"

    "project/simplebank/pb"

    "project/simplebank/util"

  

    "github.com/grpc-ecosystem/grpc-gateway/v2/runtime"

  

    api "project/simplebank/api"

    db "project/simplebank/db/sqlc"

  

    "github.com/jackc/pgx/v5/pgxpool"

    "google.golang.org/grpc"

    "google.golang.org/grpc/credentials/insecure"

    "google.golang.org/grpc/reflection"

)

  

var (

    // command-line options:

    // gRPC server endpoint

    grpcServerEndpoint = flag.String("grpc-server-endpoint", "localhost:9090", "gRPC server endpoint")

)

  

func main() {

    config, err := util.LoadConfig(".")

    if err != nil {

        log.Fatal("cannot load config:", err)

    }

  

    connPool, err := pgxpool.New(context.Background(), config.DB_SOURCE)

    if err != nil {

        log.Fatal("cannot connect to db:", err)

    }

    //初始化数据库服务

    store := db.NewStore(connPool)

    //运行gin框架

    //RunGinServer(config, store)

    go func() {

        if err := runGrpGatewayServer(config, store); err != nil {

            log.Fatal(err)

        }

  

    }()

  

    if err := runGrpcServer(config, store); err != nil {

        log.Fatalf("Failed to start gRPC server: %v", err)

    }

  

}

  

func runGrpcServer(config util.Config, store db.Store) error {

    server, err := gapi.NewServer(config, store)

    if err != nil {

        log.Fatalf("cannot create gRPCserver %v", err)

    }

    //创建gRPC服务器实例

    grpcServer := grpc.NewServer()

    //注册服务

    pb.RegisterSimplebankServer(grpcServer, server)

    //注册反射服务

    reflection.Register(grpcServer)

    //创建监听器

    listener, err := net.Listen("tcp", config.GRPCServerAddress)

    if err != nil {

        log.Fatal("cannot create Listener")

    }

    //启动gRPC服务器

    log.Printf("start gRPC server at %s", listener.Addr().String())

    // err = grpcServer.Serve(listener)

    // if err != nil {

    //  log.Fatal("cannot start gRPC server")

    // }

    return grpcServer.Serve(listener)

}

  

func runGrpGatewayServer(config util.Config, store db.Store) error {

  

    ctx, cancel := context.WithCancel(context.Background())

    defer cancel()

    grpcmux := runtime.NewServeMux()

    opts := []grpc.DialOption{grpc.WithTransportCredentials(insecure.NewCredentials())}

  

    err := pb.RegisterSimplebankHandlerFromEndpoint(ctx, grpcmux, *grpcServerEndpoint, opts)

    if err != nil {

        log.Fatalf("cannot register handler from endpoint: %v", err)

    }

    // Start HTTP server (and proxy calls to gRPC server endpoint)

    // 创建 HTTP 监听器

    listener, err := net.Listen("tcp", ":8081")

    if err != nil {

        log.Fatalf("Failed to create listener on port 8081: %v", err)

    }

  

    // 输出启动信息

    log.Printf("Starting gRPC Gateway server at %s", listener.Addr().String())

  

    return http.Serve(listener, grpcmux)

}

  

func RunGinServer(config util.Config, store db.Store) {

    server, err := api.NewServer(config, store)

    if err != nil {

        log.Fatalf("cannot create ginserver: %v", err)

    }

    err = server.Start(config.HTTPServerAddress)

    if err != nil {

        log.Fatalf("cannot start server: %v", err)

    }

}
```


#### service_simple_bank.proto:
```go
syntax = "proto3";

  

package pb;

  

import "rpc_create_user.proto";

import "rpc_login_user.proto";

import "google/api/annotations.proto";

import "protoc-gen-openapiv2/options/annotations.proto";

  

option go_package = "project/simplebank/pb";

  
  

option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_swagger) = {

  info: {

    title: "Simple Bank API"

    version: "1.1"

    contact: {

      name: "Zhaozhonghe"

      url: "https://github.com/Whuichenggong"

      email: "zhaozhonghe40@gmail.com"

    };

  };

};

  

service Simplebank {

    rpc CreateUser (CreateUserRequest) returns (CreateUserResponse) {

        option (google.api.http) = {

            post: "/v1/create_users"

            body: "*"

    };

    option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_operation) = {

      description: "User this API to create a new user"

      summary: "Create a new user"

};

}

    rpc LoginUser (LoginUserRequest) returns (LoginUserResponse) {

        option (google.api.http) = {

            post: "/v1/login_users"

            body: "*"

        };

        ;

    option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_operation) = {

      description: "User this API to login a user and get a token & refresh token"

      summary: "Login a user"

};

    }

}
```


使用了**swagger grpc gateaway**



#### **完整调用链（微服务架构）**

**如果是 HTTP 请求**

rust

复制编辑

`HTTP 请求 --> gRPC Gateway 转换为 gRPC 请求 --> gRPC Server（CreateUser）`

**如果是 gRPC 请求**

arduino

复制编辑

`gRPC 客户端 --> gRPC Server（CreateUser）`

#### protoc： 

gRPC 使用 protobuf 来定义消息格式和服务接口。protobuf 提供了数据序列化的能力，而 gRPC 在此基础上实现了高效的远程过程调用。

### 元数据

在 gRPC 和 HTTP 请求中，元数据通常指的是请求或响应中的一些附加信息，如头部信息、认证信息、客户端信息等。它通常包含在请求的上下文中，不直接参与数据传输，但对处理请求非常重要。


### 五.api 文档

#### Swagger：

```go
proto:

    del /Q pb\*.go\

    del /Q doc\swagger\*.swagger.json

    protoc --proto_path=proto --go_out=pb --go_opt=paths=source_relative \

    --go-grpc_out=pb --go-grpc_opt=paths=source_relative \

    --grpc-gateway_out=pb --grpc-gateway_opt=paths=source_relative \

    --openapiv2_out=doc/swagger --openapiv2_opt=allow_merge=true,merge_file_name=simple_bank \

    proto/*.proto
```

SwaggerHub: 与团队成员分享生成的接口

https://app.swaggerhub.com/org-575/home 导入jSON文件


生成api文档：

![[API文档.png]]




https://github.com/grpc-ecosystem/grpc-gateway/blob/main/examples/internal/proto/examplepb/a_bit_of_everything.proto

作用： 主要展示了如何使用 Protobuf 定义多种 gRPC 服务，同时如何通过 gRPC Gateway 将这些服务暴露为 RESTful API


在导入https://github.com/grpc-ecosystem/grpc-gateway/blob/main/examples/internal/proto/examplepb/a_bit_of_everything.proto 文件时产生错误：

应将https://github.com/grpc-ecosystem/grpc-gateway 仓库克隆到本地，因为我们需要仓库中的文件

protoc-gen-openapiv2

` cp *proto ..\..\..\simplebank\proto\protoc-gen-openapiv2\options\`

这样就成功导入了：

`make proto`重新生成

将会带有在service_simple_bank.proto文件中导入的配置

```go
option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_swagger) = {

  info: {

    title: "Simple Bank API"

    version: "1.0"

    contact: {

      name: "Zhaozhonghe"

      url: "https://github.com/Whuichenggong"

      email: "zhaozhonghe40@gmail.com"

    };

  };

};
```


重新生成的文档
```json
"info": {

    "title": "Simple Bank API",

    "version": "1.0",

    "contact": {

      "name": "Zhaozhonghe",

      "url": "https://github.com/Whuichenggong",

      "email": "zhaozhonghe40@gmail.com"

    }
```


	



使用 Swagger UI 免费生成

复制 Swagger UI下的dist目录到我们项目中的doc目录中

并更改url 为我们之前的生成的`simple_bank_swagger.json`

```
window.onload = function() {

  //<editor-fold desc="Changeable Configuration Block">

  

  // the following lines will be replaced by docker/configurator, when it runs in a docker-container

  window.ui = SwaggerUIBundle({

    url: "simple_bank_swagger.json",

    dom_id: '#swagger-ui',

    deepLinking: true,

    presets: [

      SwaggerUIBundle.presets.apis,

      SwaggerUIStandalonePreset

    ],

    plugins: [

      SwaggerUIBundle.plugins.DownloadUrl

    ],

    layout: "StandaloneLayout"

  });

  

  //</editor-fold>

};
```


目前的main.go
```go
package main

  

import (

    "context"

    "errors"

    "net"

    "net/http"

    "os"

    "os/signal"

    "syscall"

  

    "github.com/golang-migrate/migrate/v4"

    _ "github.com/golang-migrate/migrate/v4/database/postgres"

    _ "github.com/golang-migrate/migrate/v4/source/file"

    "github.com/grpc-ecosystem/grpc-gateway/v2/runtime"

    "github.com/hibiken/asynq"

    "github.com/jackc/pgx/v5/pgxpool"

    "github.com/rakyll/statik/fs"

    "github.com/rs/zerolog"

    "github.com/rs/zerolog/log"

    "github.com/techschool/simplebank/api"

    db "github.com/techschool/simplebank/db/sqlc"

    _ "github.com/techschool/simplebank/doc/statik"

    "github.com/techschool/simplebank/gapi"

    "github.com/techschool/simplebank/mail"

    "github.com/techschool/simplebank/pb"

    "github.com/techschool/simplebank/util"

    "github.com/techschool/simplebank/worker"

    "golang.org/x/sync/errgroup"

    "google.golang.org/grpc"

    "google.golang.org/grpc/reflection"

    "google.golang.org/protobuf/encoding/protojson"

)

  

var interruptSignals = []os.Signal{

    os.Interrupt,

    syscall.SIGTERM,

    syscall.SIGINT,

}

  

func main() {

    config, err := util.LoadConfig(".")

    if err != nil {

        log.Fatal().Err(err).Msg("cannot load config")

    }

  

    if config.Environment == "development" {

        log.Logger = log.Output(zerolog.ConsoleWriter{Out: os.Stderr})

    }

  

    ctx, stop := signal.NotifyContext(context.Background(), interruptSignals...)

    defer stop()

  

    connPool, err := pgxpool.New(ctx, config.DBSource)

    if err != nil {

        log.Fatal().Err(err).Msg("cannot connect to db")

    }

  

    runDBMigration(config.MigrationURL, config.DBSource)

  

    store := db.NewStore(connPool)

  

    redisOpt := asynq.RedisClientOpt{

        Addr: config.RedisAddress,

    }

  

    taskDistributor := worker.NewRedisTaskDistributor(redisOpt)

  

    waitGroup, ctx := errgroup.WithContext(ctx)

  

    runTaskProcessor(ctx, waitGroup, config, redisOpt, store)

    runGatewayServer(ctx, waitGroup, config, store, taskDistributor)

    runGrpcServer(ctx, waitGroup, config, store, taskDistributor)

  

    err = waitGroup.Wait()

    if err != nil {

        log.Fatal().Err(err).Msg("error from wait group")

    }

}

  

func runDBMigration(migrationURL string, dbSource string) {

    migration, err := migrate.New(migrationURL, dbSource)

    if err != nil {

        log.Fatal().Err(err).Msg("cannot create new migrate instance")

    }

  

    if err = migration.Up(); err != nil && err != migrate.ErrNoChange {

        log.Fatal().Err(err).Msg("failed to run migrate up")

    }

  

    log.Info().Msg("db migrated successfully")

}

  

func runTaskProcessor(

    ctx context.Context,

    waitGroup *errgroup.Group,

    config util.Config,

    redisOpt asynq.RedisClientOpt,

    store db.Store,

) {

    mailer := mail.NewGmailSender(config.EmailSenderName, config.EmailSenderAddress, config.EmailSenderPassword)

    taskProcessor := worker.NewRedisTaskProcessor(redisOpt, store, mailer)

  

    log.Info().Msg("start task processor")

    err := taskProcessor.Start()

    if err != nil {

        log.Fatal().Err(err).Msg("failed to start task processor")

    }

  

    waitGroup.Go(func() error {

        <-ctx.Done()

        log.Info().Msg("graceful shutdown task processor")

  

        taskProcessor.Shutdown()

        log.Info().Msg("task processor is stopped")

  

        return nil

    })

}

  

func runGrpcServer(

    ctx context.Context,

    waitGroup *errgroup.Group,

    config util.Config,

    store db.Store,

    taskDistributor worker.TaskDistributor,

) {

    server, err := gapi.NewServer(config, store, taskDistributor)

    if err != nil {

        log.Fatal().Err(err).Msg("cannot create server")

    }

  

    gprcLogger := grpc.UnaryInterceptor(gapi.GrpcLogger)

    grpcServer := grpc.NewServer(gprcLogger)

    pb.RegisterSimpleBankServer(grpcServer, server)

    reflection.Register(grpcServer)

  

    listener, err := net.Listen("tcp", config.GRPCServerAddress)

    if err != nil {

        log.Fatal().Err(err).Msg("cannot create listener")

    }

  

    waitGroup.Go(func() error {

        log.Info().Msgf("start gRPC server at %s", listener.Addr().String())

  

        err = grpcServer.Serve(listener)

        if err != nil {

            if errors.Is(err, grpc.ErrServerStopped) {

                return nil

            }

            log.Error().Err(err).Msg("gRPC server failed to serve")

            return err

        }

  

        return nil

    })

  

    waitGroup.Go(func() error {

        <-ctx.Done()

        log.Info().Msg("graceful shutdown gRPC server")

  

        grpcServer.GracefulStop()

        log.Info().Msg("gRPC server is stopped")

  

        return nil

    })

}

  

func runGatewayServer(

    ctx context.Context,

    waitGroup *errgroup.Group,

    config util.Config,

    store db.Store,

    taskDistributor worker.TaskDistributor,

) {

    server, err := gapi.NewServer(config, store, taskDistributor)

    if err != nil {

        log.Fatal().Err(err).Msg("cannot create server")

    }

  

    jsonOption := runtime.WithMarshalerOption(runtime.MIMEWildcard, &runtime.JSONPb{

        MarshalOptions: protojson.MarshalOptions{

            UseProtoNames: true,

        },

        UnmarshalOptions: protojson.UnmarshalOptions{

            DiscardUnknown: true,

        },

    })

  

    grpcMux := runtime.NewServeMux(jsonOption)

  

    err = pb.RegisterSimpleBankHandlerServer(ctx, grpcMux, server)

    if err != nil {

        log.Fatal().Err(err).Msg("cannot register handler server")

    }

  

    mux := http.NewServeMux()

    mux.Handle("/", grpcMux)

  

    statikFS, err := fs.New()

    if err != nil {

        log.Fatal().Err(err).Msg("cannot create statik fs")

    }

  

    swaggerHandler := http.StripPrefix("/swagger/", http.FileServer(statikFS))

    mux.Handle("/swagger/", swaggerHandler)

  

    httpServer := &http.Server{

        Handler: gapi.HttpLogger(mux),

        Addr:    config.HTTPServerAddress,

    }

  

    waitGroup.Go(func() error {

        log.Info().Msgf("start HTTP gateway server at %s", httpServer.Addr)

        err = httpServer.ListenAndServe()

        if err != nil {

            if errors.Is(err, http.ErrServerClosed) {

                return nil

            }

            log.Error().Err(err).Msg("HTTP gateway server failed to serve")

            return err

        }

        return nil

    })

  

    waitGroup.Go(func() error {

        <-ctx.Done()

        log.Info().Msg("graceful shutdown HTTP gateway server")

  

        err := httpServer.Shutdown(context.Background())

        if err != nil {

            log.Error().Err(err).Msg("failed to shutdown HTTP gateway server")

            return err

        }

  

        log.Info().Msg("HTTP gateway server is stopped")

        return nil

    })

}

  

func runGinServer(config util.Config, store db.Store) {

    server, err := api.NewServer(config, store)

    if err != nil {

        log.Fatal().Err(err).Msg("cannot create server")

    }

  

    err = server.Start(config.HTTPServerAddress)

    if err != nil {

        log.Fatal().Err(err).Msg("cannot start server")

    }

}
```

Swagger访问路径 ： 

http://localhost:8080/swagger/



在proto中添加描述 为 Swagger

![[SwaggerUI.png]]




### 六.gRPC自定义错误消息

![[gRPC_validate.png]]




```go
func validateCreateUserRequest(req *pb.CreateUserRequest) (violations []*errdetails.BadRequest_FieldViolation) {

    if err := val.ValidateUsername(req.GetUsername()); err != nil {

        violations = append(violations, fieldViolation("username", err))

    }

  

    if err := val.ValidatePassword(req.GetPassword()); err != nil {

        violations = append(violations, fieldViolation("password", err))

    }

  

    if err := val.ValidateFullname(req.GetFullName()); err != nil {

        violations = append(violations, fieldViolation("full_name", err))

    }

  

    if err := val.ValidateEmail(req.GetEmail()); err != nil {

        violations = append(violations, fieldViolation("email", err))

    }

  

    return violations

}
```

error.go
```go
package gapi

  

import (

    "google.golang.org/genproto/googleapis/rpc/errdetails"

    "google.golang.org/grpc/codes"

    "google.golang.org/grpc/status"

)

  

func fieldViolation(field string, err error) *errdetails.BadRequest_FieldViolation {

    return &errdetails.BadRequest_FieldViolation{

        Field:       field,

        Description: err.Error(),

    }

  

}

  

func invalidArgumentError(violations []*errdetails.BadRequest_FieldViolation) error {

    badRequest := &errdetails.BadRequest{FieldViolations: violations}

    statusInvalid := status.New(codes.InvalidArgument, "invalid parameters")

  

    statusDetails, err := statusInvalid.WithDetails(badRequest)

    if err != nil {

        return statusInvalid.Err()

    }

  

    return statusDetails.Err()

}
```


#### docker-compose up -d 和 docker-compose up 有什么区别


```
 **`docker-compose up -d`**：后台模式（Detached Mode）

- **`-d`** 代表 **detached**（后台）模式。
- 它会在后台启动所有容器，并且不会在终端输出容器的日志。
- 启动后，你可以继续使用命令行执行其他操作。
- 容器仍然在后台运行，直到你显式地停止它们。
```


```
 **`docker-compose up`**：前台模式（Foreground Mode）

- 默认情况下，`docker-compose up` 会在当前终端窗口启动服务，并且会显示容器的实时日志输出。
- 容器会持续运行，直到你手动中断（比如按 `Ctrl + C`）。
- 这种模式下，命令行会被占用，直到你停止容器。
```


### 七. 如何利用sqlc特定更新某一字段


user.sql

```sql

-- name: UpdateUser :one

UPDATE users

SET

  hashed_password = COALESCE(sqlc.narg(hashed_password), hashed_password),

  password_changed_at = COALESCE(sqlc.narg(password_changed_at), password_changed_at),

  full_name = COALESCE(sqlc.narg(full_name), full_name),

  email = COALESCE(sqlc.narg(email), email)

WHERE

  username = sqlc.arg(username)

RETURNING *;
```







### 八: 52 Write structured logs for gRPC APIs


**`zerolog`** 是一个 Go 语言的高性能、零依赖的日志库。它被设计用于高效地记录日志信息，尤其是在性能要求较高的场景下。与许多传统日志库（如 `log`）相比，`zerolog` 提供了更高效的日志记录方式，特别适用于生产环境。


官方文档:

`https://github.com/rs/zerolog`


将 log 包 换成  ”github.com/rs/zerolog/log“

将输出日志都转换为 JSON日志：

适合生产，不容易读取 开发使用人性化日志

查看官方文档：


#### Pretty logging

[](https://github.com/rs/zerolog#pretty-logging)

To log a human-friendly, colorized output, use `zerolog.ConsoleWriter`:

```go
log.Logger = log.Output(zerolog.ConsoleWriter{Out: os.Stderr})

log.Info().Str("foo", "bar").Msg("Hello world")

// Output: 3:04PM INF Hello World foo=bar
```


最终效果：

![[输出人类友好日志.png]]



```go
if config.Environment == "development" {

        log.Logger = log.Output(zerolog.ConsoleWriter{Out: os.Stderr})

    }
```
由我们自己决定日志输出什么格式

目前更改仅限于 grpc请求！

---


### 九. http拦截器中间件


关键代码:

```go


type ResponseRecorder struct {

    http.ResponseWriter

    StatusCode int

    Body       []byte

}

  

func (rec *ResponseRecorder) WriteHeader(statusCode int) {

    rec.StatusCode = statusCode

    rec.ResponseWriter.WriteHeader(statusCode)

}
```

`ResponseRecorder` 是一个自定义的类型，嵌套了 `http.ResponseWriter`，用于在 HTTP 响应过程中捕获响应的状态码（`StatusCode`）和响应体内容（`Body`）。

- **`WriteHeader(statusCode int)`**：重写了 `http.ResponseWriter` 的 `WriteHeader` 方法。正常情况下，当 `http.ResponseWriter` 被调用时，响应头会被发送。`ResponseRecorder` 拦截这个操作，记录下状态码，并将其传递给原始的 `ResponseWriter`。



---

### 新阶段： Implement background worker in Go with Redis and asynq


#### 同步api：

当客户端发送请求，服务器立刻做出响应，并返回结果同步返回给客户端

#### 异步api:



![[异步处理.png]]


缺点： 任务基本都运行在进程的memory中，一旦发生宕机，未处理数据将全部消失


![[workers.png]]

![[use redis with verification email.png]]



Asynq 是一个 Go 库，用于将任务排队并与 worker 异步处理它们。它由 [Redis](https://redis.io/) 提供支持，旨在可扩展且易于上手。:

https://github.com/hibiken/asynq


#### distribute.go 任务分发：

```go
type TaskDistributor interface {
	DistributeTaskSendVerifyEmail(
		ctx context.Context,
		payload *PayloadSendVerifyEmail,
		opts ...asynq.Option,
	) error
}
```


这个接口的作用是**抽象出任务分发的逻辑**，让具体的实现可以灵活变化，比如分发任务到异步队列。

- **`ctx context.Context`**：
    
    - 用于控制任务分发过程的上下文，可以**超时、取消**等，防止任务无限执行。
    - 常用于异步任务、网络请求等场景，确保任务能及时终止。
- **`payload *PayloadSendVerifyEmail`**：
    
    - `PayloadSendVerifyEmail` 是一个自定义结构体，包含**发送验证邮件**需要的参数（比如用户邮箱、验证码等信息）。
    - `payload` 是指针类型，表示传递结构体的地址，避免复制数据。


该方法返回一个 `error`，表示任务分发成功或失败。


**`TaskDistributor` 接口**可以让你的代码更加灵活，支持不同的任务分发实现方式，比如：

- 本地任务队列
- 使用 `asynq` 进行异步任务队列管理
- 未来如果要替换成其他任务队列系统，比如 Kafka、RabbitMQ，只需要修改实现即可，而不需要改变调用逻辑。



#### process.go

从redis队列中 选取任务对其进行处理


asynq服务器：

```go
server := asynq.NewServer(

        redisOpt,

        asynq.Config{

            Queues: map[string]int{

                QueueCritical: 10,

                QueueDefault:  5,

            },

            ErrorHandler: asynq.ErrorHandlerFunc(func(ctx context.Context, task *asynq.Task, err error) {

                log.Error().Err(err).Str("type", task.Type()).

                    Bytes("payload", task.Payload()).Msg("process task failed")

            }),

            Logger: logger,

        },

    )
```


```go
func (processor *RedisTaskProcessor) Start() error {

    mux := asynq.NewServeMux()

  

    mux.HandleFunc(TaskSendVerifyEmail, processor.ProcessTaskSendVerifyEmail)

  

    return processor.server.Start(mux)

}
```


`Start` 方法的作用是启动一个任务处理服务，使其能够接收并处理任务队列中的任务。当任务类型为 `TaskSendVerifyEmail` 时，任务将由 `ProcessTaskSendVerifyEmail` 函数处理。这个方法基本上是启动任务处理器并确保它能够实时地接收到和处理任务。



#### task_send_verify_email.go


这个方法是 `RedisTaskDistributor` 的一个实现，负责将 **"发送验证邮件"** 的任务分发到 `asynq` 队列中，并记录相关日志。

```go
  

func (distributor *RedisTaskDistributor) DistributeTaskSendVerifyEmail(

    ctx context.Context,

    payload *PayloadSendVerifyEmail,

    opts ...asynq.Option,

) error {

    jsonPayload, err := json.Marshal(payload)

    if err != nil {

        return fmt.Errorf("failed to marshal task payload: %w", err)

    }

  

    task := asynq.NewTask(TaskSendVerifyEmail, jsonPayload, opts...)

    info, err := distributor.client.EnqueueContext(ctx, task)

    if err != nil {

        return fmt.Errorf("failed to enqueue task: %w", err)

    }

  

    log.Info().Str("type", task.Type()).Bytes("payload", task.Payload()).

        Str("queue", info.Queue).Int("max_retry", info.MaxRetry).Msg("enqueued task")

    return nil

}
```


` jsonPayload, err := json.Marshal(payload)`

接收任务参数并序列化为 JSON 格式


- `payload` 是发送验证邮件任务的内容（如用户的邮箱、验证码等）。

- 该方法将 `payload` 序列化成 JSON 格式，作为任务的有效负载。

`task := asynq.NewTask(TaskSendVerifyEmail, jsonPayload, opts...)`  创建任务

` info, err := distributor.client.EnqueueContext(ctx, task)`   将任务发送到redis队列


---
反序列化和序列化在任务分发与处理中的作用不同，它们分别发生在任务的**生产**和**消费**阶段。

 为什么需要反序列化？

在 `ProcessTaskSendVerifyEmail` 方法中，我们需要从队列中取出任务，并将任务的有效载荷（`payload`）反序列化成结构体 `PayloadSendVerifyEmail`，这样才能在任务处理中使用这些数据。

具体来说：

1. **反序列化** (`json.Unmarshal`) 将任务的有效载荷从 JSON 字符串格式转换回 Go 结构体，这样你就可以在代码中更方便地使用这些数据（例如 `payload.Username`, `payload.Email`）。
    
2. 你需要这些数据来执行后续的业务逻辑，比如查找数据库中的用户、生成验证码等。所以，你必须先将其转为一个结构体，才能在代码中像操作普通对象一样访问这些数据。
    

 为什么需要序列化？

在 `DistributeTaskSendVerifyEmail` 方法中，我们将 `PayloadSendVerifyEmail` 结构体序列化成 JSON 格式，原因如下：

1. **数据传输**：任务队列中的任务实际上是在不同的系统或服务间传递的。为了使得任务能够被异步处理，我们将数据（即 `payload`）序列化为一个标准化格式（比如 JSON）。这样，无论任务队列在服务器之间如何传递，接收端都能够通过反序列化还原成原本的结构体。
    
2. **与队列系统的兼容**：任务队列（如 `asynq`）通常要求任务的数据必须是字节流（如 JSON 字符串）。因此，必须将结构体序列化成字节格式，才能通过队列传递。
    

 总结

- **序列化**：是将任务的 `Payload` 转换成 JSON 字符串，以便将其发送到队列中，进行任务的分发。
- **反序列化**：是从队列中取出任务后，将其有效载荷（JSON 格式）转换为 Go 结构体，便于在后端处理。

通过这种方式，你可以将任务内容从队列中抽象出来，然后在各个服务之间传递和处理，同时保持数据格式的标准化和一致性。

----



为什么要设置结构体为


```go
type RedisTaskProcessor struct {

    server *asynq.Server

    store  db.Store

    mailer mail.EmailSender

}
```

因为在处理过程过需要调用数据库 获取 User


**task_send_verify_email.go**

这两段函数分别实现了任务的 **分发** 和 **处理**。它们的主要作用是将任务从任务队列中推送到 Redis（或者其他任务队列）并从队列中取出任务进行处理。


**distributor.go**

负责 任务分发相关事项

**process.go**

负责 任务处理相关事项


### redis 与后端服务集成


`hub.docker.com`


~~~
docker exec -it redis redis-cli ping
 
PONG
~~~


**创建一个功能： 1. 创建文件目录， 在一个文件中定义方法，在另一个文件中实现方法，将方法注册到server中，然后主函数调用包也就是被实现的方法**


问题： 如果用户创建成功，但是发送email的任务失败了应该怎么解决呢？？？


#### db transaction

如果email任务发送失败则事物回滚

你需要将 `[]asynq.Option` 切片展开为多个 `asynq.Option` 参数。在 Go 中，可以使用 `...` 操作符来展开切片。

```go
// 使用 ... 展开切片
err := server.taskDistributor.DistributeTaskSendVerifyEmail(ctx, payload, options...)
if err != nil {
    log.Fatalf("Failed to distribute task: %v", err)
}****
```




```go
//重试次数为10 延迟处理

    options := []asynq.Option{

        asynq.MaxRetry(10),

        asynq.ProcessIn(time.Second * 5),

        asynq.Queue("critical"),

    }
```

 `asynq.Queue("critical")` 中的 `"critical"` 是一个 **队列的名称**。
- `Queue` 是一个 **选项函数（Option）**，它告诉 **asynq** 库该任务应该被放入哪个队列中。队列的目的是根据任务的不同重要性或类型来分组任务，从而使得你可以更细粒度地控制任务的执行（比如：优先级、并发数量、延迟等）。


`asynq.Queue("critical")`  将任务放进指定队列

#### 为redis设置关键队列和默认队列

设置这个之后 需要告诉信息处理者   处理“critical”这个队列消息才能从队列中取出消息然后处理 ，意味着这个任务被标记为高优先级或紧急任务，通常会被优先处理。


```go
const (

    QueueCritical = "critical"

    QueueDefault  = "default"

)
```


---

2025.3.8

如果数据库宕机了 或者 redis宕机了 如何快速的获取错误

根据asynq.Config 配置为我们提供的各种参数，


还可以自定义日志


