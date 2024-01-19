---
title: 组合模式
abbrlink: go/design/composite
date: 2021-04-03 19:11:38
tags: [go]
categories: [go]
sticky: 1
---

## 组合模式

> 定义： 将一组对象组织成树形结构，以表示一种“部分-整体”的层次结构，组合让客户端（代码使用者）可以统一单个对象和组合对象的处理逻辑
<!--more-->
> 应用场景：业务场景必须能够表示成树形结构；将一组对象组织成树形结构，将单个对象和组合对象都看做树中的节点，以统一处理逻辑，并且它利用树形结构的特点，递归处理每个子树，一次简化代码实现。

### 代码实现

```go
package design


// IOrganization 组织接口，都实现统计人数的功能
type IOrganization interface {
    Count() int
}


// Employee 员工
type Employee struct {
    Name string
}


// Count 人数统计
func (Employee) Count() int {
    return 1
}


// Department 部门
type Department struct {
    Name string
    SubOrganizations []IOrganization
}

// Count 人数统计
func (d Department) Count() int {
    c := 0
    for _, org := range d.SubOrganizations {
        c += org.Count()
    }
    return c
}

// AddSub 添加子节点
func (d *Department) AddSub(org IOrganization) {
    d.SubOrganizations = append(d.SubOrganizations, org)
}


// NewOrganization 构建组织架构 demo
func NewOrganization() IOrganization {
    root := &Department{Name: "root"}
    for i := 0; i < 10; i++ {
        root.AddSub(&Employee{})
        root.AddSub(&Department{Name: "sub", SubOrganizations: []IOrganization{&Employee{}}})
    }
    return root
}
```

### 单元测试

```go
package design

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestNewOrganization(t *testing.T) {
    got := NewOrganization().Count()
    assert.Equal(t, 20, got)
}
```
