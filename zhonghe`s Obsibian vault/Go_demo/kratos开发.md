
博客：

https://learnku.com/articles/64942


### Wire 的基本使用示例：

假设我们有一个简单的服务 `Database`，它依赖于一个配置 `Config`。我们可以使用 Wire 来自动注入 `Config` 到 `Database` 中。

- **Wire 依赖注入**简化了对象的创建和依赖关系的管理，避免了手动管理复杂的依赖链。
- Wire 会根据你定义的提供者函数自动生成依赖注入的代码，减少了手动编写和管理依赖的工作量。
- 它帮助你保持应用的松耦合和高可测试性。

Wire 主要是为了应对 Go 语言缺少内建依赖注入框架的挑战，让开发者能够以清晰和结构化的方式管理和注入依赖。




运行consul：

`docker run -d --name=consul -p 8500:8500 -e CONSUL_UI_ENABLED=true consul:1.10.0 agent -dev -client=0.0.0.0`



---
必须统一三个层次的结构

根据表结构帮我确认一下哪里有问题！其他的结构是否统一了 CREATE SCHEMA IF NOT EXISTS orders;

CREATE TABLE orders.orders
(
    id           SERIAL PRIMARY KEY,
    owner        VARCHAR(100)                  NOT NULL, -- 订单所有者
    name         VARCHAR(100)                  NOT NULL, -- 订单名称
    email        VARCHAR(100)                  NOT NULL, -- 用户邮箱
    street_address VARCHAR(255)                NOT NULL, -- 街道地址
    city         VARCHAR(100)                  NOT NULL, -- 城市
    state        VARCHAR(100)                  NOT NULL, -- 州/省
    country      VARCHAR(100)                  NOT NULL, -- 国家
    zip_code     VARCHAR(20)                   NOT NULL, -- 邮政编码
    currency     CHAR(3)                       NOT NULL, -- 货币类型, ISO 4217 标准
    status       VARCHAR(10) DEFAULT 'pending' NOT NULL, -- 订单状态: pending, paid, cancelled
    created_at   timestamptz                   NOT NULL, -- 创建时间
    updated_at   timestamptz DEFAULT now()     NOT NULL  -- 更新时间
);

CREATE TABLE orders.order_items
(
    id         SERIAL PRIMARY KEY,
    order_id   INT           NOT NULL REFERENCES orders.orders (id) ON DELETE CASCADE,
    product_id INT           NOT NULL, -- 商品ID
    name       VARCHAR(100)  NOT NULL, -- 商品名称
    quantity   INT           NOT NULL, -- 商品数量
    price      DECIMAL(10, 2) NOT NULL -- 商品单价
);
 ，，帮我根据表统一其他的，syntax = "proto3";

package order.service.v1;

import "google/api/annotations.proto";

option go_package = "api/order/v1;order";

service OrderService {
  rpc PlaceOrder(PlaceOrderReq) returns (PlaceOrderResp) {
    option (google.api.http) = {
      post: "/v1/order/place"
      body: "*"
    };
  }
  rpc ListOrder(ListOrderReq) returns (ListOrderResp) {
          option (google.api.http) = {
            get: "/v1/order/list"
        };
  }
  rpc MarkOrderPaid(MarkOrderPaidReq) returns (MarkOrderPaidResp) {
    option (google.api.http) = {
      post: "/v1/order/mark_paid"
      body: "*"
  };
  }
}

message Address {
  string street_address = 1;
  string city = 2;
  string state = 3;
  string country = 4;
  string zip_code = 5;
}

message OrderItem {
  int32 id = 1;
  string name = 2;
  string description = 3;
  float price = 4;
  int32 quantity = 5;
}

message PlaceOrderReq {
  string name = 1;
  string user_currency = 2;
  Address address = 3;
  string email = 4;
  repeated OrderItem items = 5;
  string owner = 6;
}

message OrderResult {
  int32 order_id = 1;
}

message PlaceOrderResp {
  OrderResult order = 1;
}

message ListOrderReq {
  uint32 user_id = 1;
}

message Order {
  repeated OrderItem order_items = 1;
  string order_id = 2;
  uint32 user_id = 3;
  string user_currency = 4;
  Address address = 5;
  string email = 6;
  int32 created_at = 7;
}

message ListOrderResp {
  repeated Order orders = 1;
}

message MarkOrderPaidReq {
  uint32 user_id = 1;
  string order_id = 2;
}

message MarkOrderPaidResp {}   和package biz

type PlaceOrderReq struct {
	Name         string  // 订单名称
	UserCurrency string  // 货币类型
	Address      Address // 地址信息
	Items        []Item  // 商品列表
	Email        string  // 用户邮箱
	Owner        string  // 订单所有者
}

type Address struct {
	StreetAddress string // 街道地址
	City          string // 城市
	State         string // 州/省
	Country       string // 国家
	ZipCode       string // 邮政编码
}

type Item struct {
	Id          int32   // 商品ID
	Name        string  // 商品名称
	Description string  // 商品描述
	Price       float32 // 商品单价
	Quantity    int32   // 商品数量
}

type PlaceOrderResp struct {
	Order OrderResult // 订单结果
}

type OrderResult struct {
	OrderId int32 // 订单ID
}

type ListOrderReq struct {
	UserId string // 用户ID，用于查询该用户的所有订单
}

type ListOrderResp struct {
	Orders []OrderSummary // 订单列表
}

type OrderSummary struct {
	OrderId      string  // 订单ID
	OrderName    string  // 订单名称
	OrderStatus  string  // 订单状态: pending, paid, shipped, etc.
	CreatedAt    int32   // 创建时间，使用字符串或时间戳，具体依据需求
	Address      Address // 确保这里有 Address 字段
	UserCurrency string  // 货币类型
	Email        string  // 用户邮箱
	State        string  // 订单状态
	OrderItems   []OrderItem
}

type OrderItem struct {
	Id          int32   // 商品ID
	Name        string  // 商品名称
	Description string  // 商品描述
	Price       float32 // 商品单价
	Quantity    int32   // 商品数量
}
 
---