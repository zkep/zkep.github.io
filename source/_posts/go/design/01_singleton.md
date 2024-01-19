---
title: 单例模式
abbrlink: go/design/singleton
date: 2021-03-26 18:51:02
tags: [go]
categories: [go]
sticky: 1
---


## 单例模式(Singleton Design Pattern)

单例模式采用了 饿汉式 和 懒汉式 两种实现，个人其实更倾向于饿汉式的实现，简单，并且可以将问题及早暴露，懒汉式虽然支持延迟加载，但是这只是把冷启动时间放到了第一次使用的时候，并没有本质上解决问题，并且为了实现懒汉式还不可避免的需要加锁
<!--more-->

> 饿汉模式和懒汉模式不能仅从性能比，懒汉经过第一次 Do 操作之后跟饿汉并无区别，但是懒汉模式避免了 init 这种侵入式代码，对 go test 是非常友好的

### 饿汉式

```go
package design

// Singleton 饿汉式单例
type Singleton struct{}

var singleton *Singleton

func init() {
   singleton = &Singleton{}
}

// GetInstance 获取实例
func GetInstance() *Singleton {
    return singleton
}  
```

#### 单元测试

```go
package design

import (
    "github.com/stretchr/testify/assert"
    "testing"
)

func TestGetInstance(t *testing.T) {
   assert.Equal(t, GetInstance(), GetInstance())
}

func BenchmarkGetInstanceParallel(b *testing.B) {
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            if GetInstance() != GetInstance() {
                b.Errorf("test fail")
            }
        }
    })
}
```

### 懒汉式（双重检测）

```go
package design

import "sync"

var (
    lazySingleton *Singleton
    once          = &sync.Once{}
)

func GetLazyInstance() *Singleton {
    if lazySingleton == nil {
        once.Do(func() {
        lazySingleton = &Singleton{}
        })
    }
    return lazySingleton
}
```

#### 单元测试

```go
package design

import (
    "github.com/stretchr/testify/assert"
    "testing"
)

func TestGetLazyInstance(t *testing.T) {
    assert.Equal(t, GetLazyInstance(), GetLazyInstance())
}

func BenchmarkGetLazyInstanceParallel(b *testing.B) {
    b.RunParallel(func(pb *testing.PB) {
    for pb.Next() {
            if GetLazyInstance() != GetLazyInstance() {
                b.Errorf("test fail")
            }
        }
    })
}
```
