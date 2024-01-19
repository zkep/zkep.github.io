---
title: 原型模式
abbrlink: go/design/prototype
date: 2021-03-29 22:50:02
tags: [go]
categories: [go]
sticky: 1
---


## 代码实现

* 接下来会按照一个类似课程中的例子使用深拷贝和浅拷贝结合的方式进行实现
需求: 假设现在数据库中有大量数据，包含了关键词，关键词被搜索的次数等信息，模块 A 为了业务需要
  * 会在启动时加载这部分数据到内存中
  * 并且需要定时更新里面的数据
  * 同时展示给用户的数据每次必须要是相同版本的数据，不能一部分数据来自版本 1 一部分来自版本 2

```go
package design

import (
    "encoding/json"
    "time"
)

type Keyword struct {
    word   string
    visit  int
    UpdatedAt *time.Time
}

// Clone 这里使用序列化与反序列化的方式深拷贝
func (k *Keyword) Clone() *Keyword {
    var newKeyword Keyword
    b, _ := json.Marshal(k)
    json.Unmarshal(b, &newKeyword)
    return &newKeyword
}

type Keywords map[string]*Keyword

// Clone 复制一个新的 keywords
// updatedWords: 需要更新的关键词列表，由于从数据库中获取数据常常是数组的方式
func (words Keywords) Clone(updateWords []*Keyword) Keywords {
    newKeywords := Keywords{}
    for k, v := range words {
        // 这里是浅拷贝，直接拷贝了地址
        newKeywords[k] = v
    }
    // 替换需要更新的字段，这里用的深拷贝
    for _, word := range updateWords {
        newKeywords[word.word] = word.Clone()
    }
    return newKeywords
}
```

### 单元测试

```go
package design

import (
    "testing"
    "time"

    "github.com/stretchr/testify/assert"
)

func TestKeywords_Clone(t *testing.T) {
    updateAt, _ := time.Parse("2006", "2020")
    words := Keywords{
        "testA": &Keyword{
        word:      "testA",
        visit:     1,
        UpdatedAt: &updateAt,
    },
    "testB": &Keyword{
        word:      "testB",
        visit:     2,
        UpdatedAt: &updateAt,
    },
    "testC": &Keyword{
        word:      "testC",
        visit:     3,
        UpdatedAt: &updateAt,
    },
    }

    now := time.Now()
    updatedWords := []*Keyword{
        {
            word:      "testB",
            visit:     10,
            UpdatedAt: &now,
        },
    }

    got := words.Clone(updatedWords)

    assert.Equal(t, words["testA"], got["testA"])
    assert.NotEqual(t, words["testB"], got["testB"])
    assert.NotEqual(t, updatedWords[0], got["testB"])
    assert.Equal(t, words["testC"], got["testC"])
}
```
