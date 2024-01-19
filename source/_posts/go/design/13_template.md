---
title: 模版模式
abbrlink: go/design/template
date: 2021-04-08 19:01:38
tags: [go]
categories: [go]
sticky: 1
---

## 模版模式

>定义：模版模式在一个方法中定义一个算法骨架，并将某些步骤推迟到子类中实现，模版模式可以让子类在不改变算法整体结构的情况下，重新定义算法中的某些步骤
>应用场景：扩展，框架通过模版模式提供功能扩展点，让框架用户可以在不修复爱框架源码的情况下，基于扩展点定制化框架的功能；复用，复用是指所有子类可以复用父类中提供的模版方法的代码。
<!--more-->
与回调的区别：同步回调，回调基于组合关系来实现，把一个对象传递给另一个对象，是一种对象之间的关系。模版模式基于继承关系来实现，子类重写父类的抽象方法，是一种类之间的关系；异步回调，更类似观察者模式。

### 代码实现

```go
package design

import "fmt"

type ISMS interface {
    send(content string, phone int) error
}

type sms struct {
    ISMS
}

func (s *sms) Valid(content string) error {
    if len(content) > 63 {
        return fmt.Errorf("content is too long")
    }
    return nil
}

func (s *sms) Send(content string, phone int) error {
    if err := s.Valid(content); err != nil {
        return err
    }
    return s.send(content, phone)
}

type TelecomSms struct {
    *sms
}

func NewTelecomSms() *TelecomSms {
    tel := &TelecomSms{}
    tel.sms = &sms{ISMS: tel}
    return tel
}

func (tel *TelecomSms) send(content string, phone int) error {
    fmt.Println("send by telecom success")
    return nil
}

```

### 单元测试

```go
package design

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func Test_sms_Send(t *testing.T) {
    tel := NewTelecomSms()
    err := tel.Send("test", 123456)
    assert.NoError(t, err)
}

```
