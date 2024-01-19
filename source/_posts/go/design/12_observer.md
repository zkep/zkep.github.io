---
title: 观察者模式
abbrlink: go/design/observer
date: 2021-04-05 18:10:38
tags: [go]
categories: [go]
sticky: 1
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

## 使用 Golang 实现 EventBus

我们实现一个支持以下功能的事件总线

+ 异步不阻塞
+ 支持任意参数值

### 代码实现

```go
import (
    "fmt"
    "reflect"
    "sync"
)

type Bus interface {
    Subscribe(topic string, handelr interface{}) error
    Publish(topic string, args ...interface{})
}

type AsyncEventBus struct {
    handlers map[string][]reflect.Value
    lock     sync.Mutex
}

func NewAsyncEventBus() *AsyncEventBus {
    return &AsyncEventBus{
        handlers: map[string][]reflect.Value{},
        lock:     sync.Mutex{},
    }
}

func (self *AsyncEventBus) Subscribe(topic string, f interface{}) error {
    self.lock.Lock()
    defer self.lock.Unlock()

    v := reflect.ValueOf(f)
    if v.Type().Kind() != reflect.Func {
        return fmt.Errorf("handler is not a function")
    }

    handler, ok := self.handlers[topic]
    if !ok {
        handler = []reflect.Value{}
    }
    handler = append(handler, v)
    self.handlers[topic] = handler

    return nil
}

func (self *AsyncEventBus) Publish(topic string, args ...interface{}) {
    handlers, ok := self.handlers[topic]
    if !ok {
        fmt.Println("not found handlers in topic:", topic)
        return
    }
    params := make([]reflect.Value, len(args))

    for i, arg := range args {
        params[i] = reflect.ValueOf(arg)
    }

    for i := range handlers {
        go handlers[i].Call(params)
    }

    return
}

```

### 单元测试

```go
package design

import (
    "fmt"
    "testing"
    "time"
)

func TestAsyncEventBus(t *testing.T) {
    bus := NewAsyncEventBus()
    bus.Subscribe("topic:1", subOne)
    bus.Subscribe("topic:2", subTwo)
    bus.Publish("topic:1", "test1", "test2")
    bus.Publish("topic:1", "testA", "testB")
    time.Sleep(1 * time.Second)
}

func subOne(msgs ...string) {
    time.Sleep(1 * time.Microsecond)
    fmt.Printf("subOne,%+v", msgs)
}

func subTwo(msgs ...string) {
    fmt.Printf("subTwo,%+v", msgs)
}

```
