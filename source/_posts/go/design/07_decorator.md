---
title: 装饰器模式
abbrlink: go/design/decorator
date: 2021-04-01 21:05:02
tags: [go]
categories: [go]
sticky: 1
---

## 装饰器模式

> 装饰器模式主要解决继承关系过于复杂的问题，通过组合来替代继承，它主要的作用是给原始类添加增强功能
<!--more-->

> 实现方式 分为接口 ，继承。如果装饰器类的方法大部分和源类相同，不做改变选继承

> 和代理模式的区别 ，代码形式上几乎没有什么差别，代理模式主要给原始类添加无关的功能
> 装饰器模式，主要是给原始类增强功能，添加的功能都是有关联的

### 代码实现

下面是一个简单的画画的例子，默认的Square只有基础的画画功能，ColorSquare 为他加上了颜色

```go
package design

type IDraw interface {
     Draw() string
}

type Square struct {}

func (s Square) Draw() string{
    return "this is a square"
}


type ColorSquare struct {
    square IDraw
    color string
}

func NewColorSquare(square IDraw,color string) ColorSquare{
    return ColorSquare{square: square,color: color}
}

func (c ColorSquare) Draw() string{
    return  c.square.Draw() +", color is " +c.color
}

```

### 单元测试

```go
package design

import (
    "testing"

    "github.com/stretchr/testify/assert"
)


func TestColorSquare_Draw(t *testing.T) {
    sq := Square{}
    csq := NewColorSquare(sq, "red")
    got := csq.Draw()
    assert.Equal(t, "this is a square, color is red", got)
}
```
