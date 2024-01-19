---
title: 策略模式
abbrlink: go/design/strategy
date: 2021-04-09 19:51:02
tags: [go]
categories: [go]
sticky: 1
---

## 策略模式

> 定义： 定义一组算法类，将每个算法分别封装起来，让他们可以互相替换，策略模式可以使算法的变化独立于使用它们的客户端。

<!--more-->

### 代码实现

```go
package strategy

import (
	"fmt"
	"io/ioutil"
	"os"
)

// StorageStrategy 存储策略
type StorageStrategy interface {
	Save(name string, data []byte) error
}

var strategys = map[string]StorageStrategy{
	"file":         &fileStorage{},
	"encrypt_file": &encryptFileStorage{},
}

// NewStorageStrategy NewStorageStrategy
func NewStorageStrategy(t string) (StorageStrategy, error) {
	s, ok := strategys[t]
	if !ok {
		return nil, fmt.Errorf("not found StorageStrategy: %s", t)
	}

	return s, nil
}

// FileStorage 保存到文件
type fileStorage struct{}

// Save Save
func (s *fileStorage) Save(name string, data []byte) error {
	return ioutil.WriteFile(name, data, os.ModeAppend)
}

// encryptFileStorage 加密保存到文件
type encryptFileStorage struct{}

// Save Save
func (s *encryptFileStorage) Save(name string, data []byte) error {
	// 加密
	data, err := encrypt(data)
	if err != nil {
		return err
	}

	return ioutil.WriteFile(name, data, os.ModeAppend)
}

func encrypt(data []byte) ([]byte, error) {
	// 这里实现加密算法
	return data, nil
}

```

### 单元测试

```go
package strategy

import (
	"testing"

	"github.com/stretchr/testify/assert"
)

func Test_demo(t *testing.T) {
	// 假设这里获取数据，以及数据是否敏感
	data, sensitive := getData()
	strategyType := "file"
	if sensitive {
		strategyType = "encrypt_file"
	}

	storage, err := NewStorageStrategy(strategyType)
	assert.NoError(t, err)
	assert.NoError(t, storage.Save("./test.txt", data))
}

// getData 获取数据的方法
// 返回数据，以及数据是否敏感
func getData() ([]byte, bool) {
	return []byte("test data"), false
}

```
