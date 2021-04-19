---
title: 观察者模式
abbrlink: observer
date: 2021-04-05 18:10:38
tags: [go]
---



## 观察者模式

> 别名：发布订阅模式
> 定义： 在对象之间定一个一对多的依赖，当一个对象的状态改变的时候，所有依赖的对象都会自动收到通知
> 实现方式： 同步阻塞；异步非阻塞（使用协程）；同进程；跨进程（消息队列）
<!--more-->

### 代码实现

```go
package design

import "fmt"

type IObserver interface {
    Update(msg string)
}

type Subject struct {
    observers []IObserver
}

func (self *Subject) Register(observer IObserver) {
    self.observers = append(self.observers, observer)
}

func (self *Subject) Remove(observer IObserver) {
    for i, obj := range self.observers {
        if obj == observer {
            self.observers = append(self.observers[:i], self.observers[i+1:]...)
        }
    }
}

func (self *Subject) Notify(msg string) {
    for _, o := range self.observers {
        o.Update(msg)
    }
}

type ObserverOne struct{}

func (obj *ObserverOne) Update(msg string) {
    fmt.Printf("ObserverOne:%s", msg)
}

type ObserverTwo struct{}

func (obj *ObserverTwo) Update(msg string) {
    fmt.Printf("ObserverTwo:%s", msg)
}

```

### 单元测试

```go
package design

import "testing"

func TestSubject_Notify(t *testing.T) {
    sub := &Subject{}
    sub.Register(&ObserverOne{})
    sub.Register(&ObserverTwo{})
    sub.Notify("hi")
}

```
