---
title: 门面模式
abbrlink: go/design/facade
date: 2021-04-02 23:11:38
tags: [go]
categories: [go]
sticky: 1
---

## 门面模式

> 别名：为外观模式
> 定义：门面模式为子系统提供统一的接口，定义一组高层接口让子系统更易用；将几个细粒度的接口包装一下，成一个接口
<!--more-->
> 应用场景：解决易用性问题，用来封装系统底层实现，隐藏系统的复杂性，提供一组更加简单易用，更高层的接口；解决性能问题，减少网络请求；解决分布式事务问题，不用多次调用了，一次调用就可以同一个进程事务中解决；

### 代码实现

```go
package design

//  IUserinfo 用户接口
type IUserInfo interface {
    Login(phone, code int) (*UserInfo, error)
    Register(phone, code int) (*UserInfo, error)
}

type UserInfo struct {
    Name string
}

// IUserInfoFacade门面模式
type IUserInfoFacade interface {
    LoginOrRegister(phone, code int) (*UserInfo, error)
}

type UserService struct{}

func (u UserService) Login(phone, code int) (*UserInfo, error) {
    return &UserInfo{Name: "login"}, nil
}

func (u UserService) Register(phone, code int) (*UserInfo, error) {
    return &UserInfo{Name: "register"}, nil
}

func (u UserService) LoginOrRegister(phone, code int) (*UserInfo, error) {
    user, err := u.Login(phone, code)
    if err != nil {
         return nil, err
    }
    if user != nil {
        return user, nil
    }
    return u.Register(phone, code)
}

```

### 单元测试

```go
package design

import (
    "github.com/stretchr/testify/assert"
    "testing"
)

func TestUserService_Login(t *testing.T) {
    service := UserService{}
    user, err := service.Login(10506000010, 1234)
    assert.NoError(t, err)
    assert.Equal(t, &UserInfo{Name: "login"}, user)
}

func TestUserService_LoginOrRegister(t *testing.T) {
    service := UserService{}
    user, err := service.LoginOrRegister(10506000010, 1234)
    assert.NoError(t, err)
    assert.Equal(t, &UserInfo{Name: "login"}, user)
}

func TestUserService_Register(t *testing.T) {
    service := UserService{}
    user, err := service.Register(10506000010, 1234)
    assert.NoError(t, err)
    assert.Equal(t, &UserInfo{Name: "register"}, user)
}
```
