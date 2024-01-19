---
title: 桥接模式
abbrlink: go/design/bridge
date: 2021-03-31 21:39:02
tags: [go]
categories: [go]
sticky: 1
---


## 桥接模式

> 将抽象和实现解藕，让它们可以独立变化
> 一个类存在两个或多个独立变化的纬度，我们通过组合的方式，让两个或多个纬度可以独立进行扩展
<!--nore-->

> 栗子，监控告警，有不同的告警类别，有不同的通知类型
> 将通知类型和告警类别进行拆分成两个类，将通知类型作为参数传递即可。
> 很多设计模式都是试图将庞大的类拆分成更小的类，然后再通过某种更合理的结构组装在一起

### 代码实现

```go
package design

// 桥接模式

type IMsgSender interface {
    Send(msg string) error
}

type EmailMsgSender struct {
    emails []string
}

func NewEmailMsgSender(emails []string) *EmailMsgSender {
    return &EmailMsgSender{emails: emails}
}

func (s *EmailMsgSender) Send(msg string) error {
    return nil
}

// 通知接口
type INotification interface {
    Notify(msg string) error
}

// 错误通知
type ErrorNotification struct {
    sender IMsgSender
}

func NewErrorNotification(sender IMsgSender) *ErrorNotification {
     return &ErrorNotification{sender: sender}
}

func (n *ErrorNotification) Notify(msg string) error {
    return n.sender.Send(msg)
}

```

### 单元测试

```go
package design

import (
    "testing"
    "github.com/stretchr/testify/assert"
)


func TestErrorNotification_Notify(t *testing.T) {
    sender := NewEmailMsgSender([]string{"test@test.com"})
    n := NewErrorNotification(sender)
    err := n.Notify("test msg")
    assert.Nil(t, err)
}
```
