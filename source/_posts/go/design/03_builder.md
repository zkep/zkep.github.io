---
title: 建造者模式
abbrlink: go/design/builder
date: 2021-03-28 18:51:02
tags: [go]
categories: [go]
sticky: 1
---

> 其实在 Golang 中对于创建类参数比较多的对象的时候，我们常见的做法是必填参数直接传递，可选参数通过传递可变的方法进行创建。
<!--more-->

### 建造者模式

  通过下面可以看到，使用 Go 编写建造者模式的代码其实会很长，这些是它的一个缺点，所以如果不是参数的校验逻辑很复杂的情况下一般我们在 Go 中不会采用这种方式，而会采用后面的另外一种方式

#### 代码

```go
package design

import "fmt"

const (
    defaultMaxTotal = 10
    defaultMaxIdle  = 9
    defaultMinIdle  = 1
)

// ResourcePoolConfig resource pool
type ResourcePoolConfig struct {
    name     string
    maxTotal int
    maxIdle  int
    minIdle  int
}

// ResourcePoolConfigBuilder 用于构建 ResourcePoolConfig
type ResourcePoolConfigBuilder struct {
    name     string
    maxTotal int
    maxIdle  int
    minIdle  int
}

// SetName SetName
func (b *ResourcePoolConfigBuilder) SetName(name string) error {
    if name == "" {
        return fmt.Errorf("name can not be empty")
    }
    b.name = name
    return nil
}

// SetMinIdle SetMinIdle
func (b *ResourcePoolConfigBuilder) SetMinIdle(minIdle int) error {
    if minIdle < 0 {
        return fmt.Errorf("max tatal cannot < 0, input: %d", minIdle)
    }
    b.minIdle = minIdle
    return nil
}

// SetMaxIdle SetMaxIdle
func (b *ResourcePoolConfigBuilder) SetMaxIdle(maxIdle int) error {
    if maxIdle < 0 {
        return fmt.Errorf("max tatal cannot < 0, input: %d", maxIdle)
    }
    b.maxIdle = maxIdle
    return nil
}

// SetMaxTotal SetMaxTotal
func (b *ResourcePoolConfigBuilder) SetMaxTotal(maxTotal int) error {
    if maxTotal <= 0 {
        return fmt.Errorf("max tatal cannot <= 0, input: %d", maxTotal)
    }
    b.maxTotal = maxTotal
    return nil
}

// Build Build
func (b *ResourcePoolConfigBuilder) Build() (*ResourcePoolConfig, error) {
    if b.name == "" {
        return nil, fmt.Errorf("name can not be empty")
    }

    // 设置默认值
    if b.minIdle == 0 {
        b.minIdle = defaultMinIdle
    }

    if b.maxIdle == 0 {
        b.maxIdle = defaultMaxIdle
    }

    if b.maxTotal == 0 {
        b.maxTotal = defaultMaxTotal
    }

    if b.maxTotal < b.maxIdle {
        return nil, fmt.Errorf("max total(%d) cannot < max idle(%d)", b.maxTotal, b.maxIdle)
    }

    if b.minIdle > b.maxIdle {
        return nil, fmt.Errorf("max idle(%d) cannot < min idle(%d)", b.maxIdle, b.minIdle)
    }

    return &ResourcePoolConfig{
        name:     b.name,
        maxTotal: b.maxTotal,
        maxIdle:  b.maxIdle,
        minIdle:  b.minIdle,
     }, nil
}

```

#### 单元测试

```go
package design

import (
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestResourcePoolConfigBuilder_Build(t *testing.T) {
 tests := []struct {
        name    string
        builder *ResourcePoolConfigBuilder
        want    *ResourcePoolConfig
        wantErr bool
    }{
    {
        name: "name empty",
        builder: &ResourcePoolConfigBuilder{
            name:     "",
            maxTotal: 0,
        },
        want:    nil,
        wantErr: true,
    },
    {
        name: "maxIdle < minIdle",
        builder: &ResourcePoolConfigBuilder{
            name:     "test",
            maxTotal: 0,
            maxIdle:  10,
            minIdle:  20,
        },
        want:    nil,
        wantErr: true,
    },
    {
        name: "success",
        builder: &ResourcePoolConfigBuilder{
            name: "test",
        },
        want: &ResourcePoolConfig{
            name:     "test",
            maxTotal: defaultMaxTotal,
            maxIdle:  defaultMaxIdle,
            minIdle:  defaultMinIdle,
        },
        wantErr: false,
    },
 }

 for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got, err := tt.builder.Build()
        require.Equalf(t, tt.wantErr, err != nil, "Build() error = %v, wantErr %v", err, tt.wantErr)
        assert.Equal(t, tt.want, got)
    })
  }
}
```

### Go 常用的参数传递方

#### 代码

```go

// ResourcePoolConfigOption option
type ResourcePoolConfigOption struct {
    maxTotal int
    maxIdle  int
    minIdle  int
}

// ResourcePoolConfigOptFunc to set option
type ResourcePoolConfigOptFunc func(option *ResourcePoolConfigOption)

// NewResourcePoolConfig NewResourcePoolConfig
func NewResourcePoolConfig(name string, opts ...ResourcePoolConfigOptFunc) (*ResourcePoolConfig, error) {
    if name == "" {
        return nil, fmt.Errorf("name can not be empty")
    }

    option := &ResourcePoolConfigOption{
            maxTotal: 10,
            maxIdle:  9,
            minIdle:  1,
         }

    for _, opt := range opts {
        opt(option)
    }

    if option.maxTotal < 0 || option.maxIdle < 0 || option.minIdle < 0 {
        return nil, fmt.Errorf("args err, option: %v", option)
    }

    if option.maxTotal < option.maxIdle || option.minIdle > option.maxIdle {
        return nil, fmt.Errorf("args err, option: %v", option)
    }

    return &ResourcePoolConfig{
        name:     name,
        maxTotal: option.maxTotal,
        maxIdle:  option.maxIdle,
        minIdle:  option.minIdle,
        }, nil
}

```

#### 单元测试

```go
package design

import (
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestNewResourcePoolConfig(t *testing.T) {
    type args struct {
        name string
        opts []ResourcePoolConfigOptFunc
    }
    tests := []struct {
        name    string
        args    args
        want    *ResourcePoolConfig
        wantErr bool
    }{
    {
        name: "name empty",
        args: args{
            name: "",
        },
        want:    nil,
        wantErr: true,
    },
    {
        name: "success",
        args: args{
            name: "test",
            opts: []ResourcePoolConfigOptFunc{
                func(option *ResourcePoolConfigOption) {
                    option.minIdle = 2
                },
            },
        },
        want: &ResourcePoolConfig{
            name:     "test",
            maxTotal: 10,
            maxIdle:  9,
            minIdle:  2,
        },
        wantErr: false,
      },
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := NewResourcePoolConfig(tt.args.name, tt.args.opts...)
            require.Equalf(t, tt.wantErr, err != nil, "error = %v, wantErr %v", err, tt.wantErr)
            assert.Equal(t, tt.want, got)
        })
    }
}
```

### 总结

其实可以看到，绝大多数情况下直接使用后面的这种方式就可以了，并且在编写公共库的时候，强烈建议入口的参数都可以这么传递，这样可以最大程度的保证我们公共库的兼容性，避免在后续的更新的时候出现破坏性的更新的情况。
