---
title: 适配器模式
abbrlink: go/design/adapter
date: 2021-04-01 21:01:02
tags: [go]
categories: [go]
sticky: 1
---


## 适配器模式

> 顾名思义，这个模式就是用来适配的，它将不兼容的接口转为可以兼容的接口，让原本不兼容而不能一起工作的类可以一起工作

> 实现方式：类适配器通过继承实现；对象适配器通过组合实现
> 如何选择：需要适配的接口不多，两种都可以。需要适配的接口很多的情况下，如果还需要大量修改选用对象适配器，不需要大量修改选用类适配器

> 应用场景：接口不兼容的情况下，封装有缺陷的接口设计；统一多个类的接口设计；替换依赖的外部系统；兼容老版本接口；适配不同格式的数据。

### 代码实现

```go
package design

// 适配器模式

type ICreateServer interface {
    CreateServer(cpu,mem float64)error
}


type QiNiuClient struct {}

func(c *QiNiuClient) RunInstance(cpu,mem float64) error{
    return nil
}

type QiniuClientAdapter struct {
    Client QiNiuClient
}

func(q *QiniuClientAdapter) CreateServer(cpu,mem float64) error{
     return q.Client.RunInstance(cpu,mem)
}

type AliYunClient struct {}

func(c *AliYunClient) CreateServer(cpu,mem float64) error{
     return nil
}

type AliYunClientAdapter struct {
 Client AliYunClient
}

func(a *AliYunClientAdapter) CreateServer(cpu,mem float64) error{
    return a.Client.CreateServer(cpu,mem)
}
```

### 单元测试

```go
package design


import (
    "testing"
)

func TestAliyunClientAdapter_CreateServer(t *testing.T) {
    var a ICreateServer = &AliYunClientAdapter{
    Client: AliYunClient{},
    }
    a.CreateServer(1.0, 2.0)
}

func TestQiNiuClientAdapter_CreateServer(t *testing.T) {
    var a ICreateServer = &QiNiuClientAdapter{
    Client: QiNiuClient{},
    }

    a.CreateServer(1.0, 2.0)
}

```
