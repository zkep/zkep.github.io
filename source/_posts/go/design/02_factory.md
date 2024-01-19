---
title: 工厂模式
abbrlink: go/design/factory
date: 2021-03-27 18:51:02
tags: [go]
categories: [go]
sticky: 1
---


### 简单工厂

由于 Go 本身是没有构造函数的，一般而言我们采用 NewName 的方式创建对象/接口，当它返回的是接口的时候，其实就是简单工厂模式

<!--more-->

```go
// 简单工厂
type IRuleConfigParser interface {
     Pasre(data []byte)
}

// jsonRuleConfigParser
type jsonRuleConfigParser struct{}


func(J jsonRuleConfigParser) Pasre(data []byte) {
     panic("implement me")
}

// yamlRuleConfigParser
type yamlRuleConfigParser struct{}


func(Y yamlRuleConfigParser) Pasre(data []byte) {
    panic("implement me")
}


func NewIRuleConfigParser(t string) IRuleConfigParser{
    switch t {
        case "json":
            return jsonRuleConfigParser{}
        case "yaml":
            return yamlRuleConfigParser{}
    }
    return nil
}


```

#### 单元测试

```go
package design

import (
    "reflect"
    "testing"
)

func TestNewIRuleConfigParser(t *testing.T) {
 type args struct {
     t string
 }
 tests := []struct {
        name string
        args args
        want IRuleConfigParser
     }{
        {
            name: "json",
            args: args{t: "json"},
            want: jsonRuleConfigParser{},
        },
        {
            name: "yaml",
            args: args{t: "yaml"},
            want: yamlRuleConfigParser{},
        },
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := NewIRuleConfigParser(tt.args.t); !reflect.DeepEqual(got, tt.want) {
                t.Errorf("NewIRuleConfigParser() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

### 工厂方法

```go
// 工厂方法

type IRuleConfigParserFactory interface{
    CreateParser() IRuleConfigParser
}


type jsonRuleConfigParserFactory struct {
}


func(J jsonRuleConfigParserFactory) CreateParser() IRuleConfigParser{
    return jsonRuleConfigParser{}
}



type yamlRuleConfigParserFactory struct {
}


func(Y yamlRuleConfigParserFactory) CreateParser() IRuleConfigParser{
    return yamlRuleConfigParser{}
}

func NewIRuleConfigParserFactory(t string) IRuleConfigParserFactory{
    switch t {
        case "json":
            return jsonRuleConfigParserFactory{}
        case "yaml":
            return yamlRuleConfigParserFactory{}
    }
    return nil
}


```

#### 单元测试

```go

func TestNewIRuleConfigParserFactory(t *testing.T) {
    type args struct {
        t string
    }
 tests := []struct {
        name string
        args args
        want IRuleConfigParserFactory
     }{
        {
            name: "json",
            args: args{t: "json"},
            want: jsonRuleConfigParserFactory{},
        },
        {
            name: "yaml",
            args: args{t: "yaml"},
            want: yamlRuleConfigParserFactory{},
        },
  }
  for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := NewIRuleConfigParserFactory(tt.args.t); !reflect.DeepEqual(got, tt.want) {
                t.Errorf("NewIRuleConfigParserFactory() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

### 抽象工厂

```go

type IRuleConfigParser interface {
     Pasre(data []byte)
}

// jsonRuleConfigParser
type jsonRuleConfigParser struct{}


func(J jsonRuleConfigParser) Pasre(data []byte) {
     panic("implement me")
}


type ISystemConfigParser interface {
    ParseSystem(data []byte)
}

type jsonSystemConfigParser struct{}

func (j jsonSystemConfigParser) ParseSystem(data []byte) {
    panic("implement me")
}

type IConfigParserFactory interface {
    CreateRuleParser() IRuleConfigParser
    CreateSystemParser() ISystemConfigParser
}

type jsonConfigParserFactory struct{}

func (J jsonConfigParserFactory) CreateRuleParser() IRuleConfigParser {
    return jsonRuleConfigParser{}
}

func (J jsonConfigParserFactory) CreateSystemParser() ISystemConfigParser {
    return jsonSystemConfigParser{}
}

```

#### 单元测试

```go

func Test_jsonConfigParserFactory_CreateRuleParser(t *testing.T) {
    tests := []struct {
            name string
            want IRuleConfigParser
        }{
        {
            name: "json",
            want: jsonRuleConfigParser{},
        },
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            j := jsonConfigParserFactory{}
            if got := j.CreateRuleParser(); !reflect.DeepEqual(got, tt.want) {
                t.Errorf("CreateRuleParser() = %v, want %v", got, tt.want)
            }
        })
    }
 }

func Test_jsonConfigParserFactory_CreateSystemParser(t *testing.T) {
    tests := []struct {
            name string
            want ISystemConfigParser
        }{
        {
            name: "json",
            want: jsonSystemConfigParser{},
        },
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            j := jsonConfigParserFactory{}
            if got := j.CreateSystemParser(); !reflect.DeepEqual(got, tt.want) {
                t.Errorf("CreateSystemParser() = %v, want %v", got, tt.want)
            }
        })
    }
}

```

### DI 容器

golang 现有的依赖注入框架:

* 使用反射实现的: <https://github.com/uber-go/dig>
* 使用 generate 实现的: <https://github.com/google/wire>
下文将通过反射实现一个类似 dig 简单的 demo，在课程里面的例子是读取配置文件，然后进行生成，下面提供是通过 provider 进行构建依赖关系。

```go

// Container DI 容器
type Container struct {
    // 假设一种类型只能有一个 providor 提供
    providers map[reflect.Type]provider

    //  缓存以生成的对象
    results map[reflect.Type]reflect.Value
}

type provider struct {
    value  reflect.Value
    params []reflect.Type
}

// 创建容器
func New() *Container {
    return &Container{
        providers: map[reflect.Type]provider{},
        results:    map[reflect.Type]reflect.Value{},
    }
}

func isError(t reflect.Type) bool {
    if t.Kind() != reflect.Interface {
        return false
    }
    return t.Implements(reflect.TypeOf(reflect.TypeOf((*error)(nil)).Elem()))
}

// Provide 对象提供者，需要传入一个对象的工厂方法，后续会用于对象的创建

func (c *Container) Provide(constructor interface{}) error {
    v := reflect.ValueOf(constructor)

    if v.Kind() != reflect.Func {
        return fmt.Errorf("constructor must be a func")
    }

    vt := v.Type()

    // 获取参数
    params := make([]reflect.Type, vt.NumIn())
    for i := 0; i < vt.NumIn(); i++ {
         params[i] = vt.In(i)
    }

    // 获取返回值
    results := make([]reflect.Type, vt.NumOut())
    for i := 0; i < vt.NumOut(); i++ {
        results[i] = vt.Out(i)
    }

    provider := provider{ value: v, params: params}

    // 保存不同类型的provider
    for _, result := range results {
        // 判断返回值error
        if isError(result) {
            continue
        }
        if _, ok := c.providers[result]; ok {
            return fmt.Errorf("%s had a provider", result)
        }
        c.providers[result] = provider
    }
    return nil
}

// Invoke 函数执行入口
func (c *Container) Invoke(function interface{}) error {
    v := reflect.ValueOf(function)
    // 仅支持函数 provider
    if v.Kind() != reflect.Func {
        return fmt.Errorf("constructor must be a func")
    }

    vt := v.Type()

    var err error
    params:= make([]reflect.Value,vt.NumIn())
    for i:=0;i<vt.NumIn();i++{
        params[i], err = c.buildParam(vt.In(i))
        if err != nil {
             return err
        }
    }
    v.Call(params)

    return nil
}

// buildParam 构建参数
// 1. 从容器中获取 provider
// 2. 递归获取 provider 的参数值
// 3. 获取到参数之后执行函数
// 4. 将结果缓存并且返回结果
func (c *Container) buildParam(param reflect.Type)(val reflect.Value, err error){
    if result, ok := c.results[param]; ok {
        return result, nil
    }
    provider, ok := c.providers[param]
    if !ok {
        return reflect.Value{}, fmt.Errorf("can not found provider: %s", param)
    }
    params := make([]reflect.Value, len(provider.params))
    for i, p := range provider.params {
        params[i], err = c.buildParam(p)
    }
    results := provider.value.Call(params)
    for _, result := range results {
        // 判断是否报错
        if isError(result.Type()) && !result.IsNil() {
            return reflect.Value{}, fmt.Errorf("%s call err: %+v", provider, result)
        }
        if !isError(result.Type()) && !result.IsNil() {
            c.results[result.Type()] = result
        }
    }
    return c.results[param], nil
}

```
