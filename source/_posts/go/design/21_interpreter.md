---
title: 解释器模式
abbrlink: go/design/interpreter
date: 2021-04-16 20:24:02
tags: [go]
categories: [go]
sticky: 1
---

## 解释器模式
> 定义：为某个语言定义它的语法表示，并定义一个解释器来处理这个语法
<!--more-->


### 代码实现

```go
// Package interpreter 解释器模式
// 采用原课程的示例, 并且做了一下简化
// 假设我们现在有一个监控系统
// 现在需要实现一个告警模块，可以根据输入的告警规则来决定是否触发告警
// 告警规则支持 &&、>、< 3种运算符
// 其中 >、< 优先级比  && 更高
package interpreter

import (
	"fmt"
	"regexp"
	"strconv"
	"strings"
)

// AlertRule 告警规则
type AlertRule struct {
	expression IExpression
}

// NewAlertRule NewAlertRule
func NewAlertRule(rule string) (*AlertRule, error) {
	exp, err := NewAndExpression(rule)
	return &AlertRule{expression: exp}, err
}

// Interpret 判断告警是否触发
func (r AlertRule) Interpret(stats map[string]float64) bool {
	return r.expression.Interpret(stats)
}

// IExpression 表达式接口
type IExpression interface {
	Interpret(stats map[string]float64) bool
}

// GreaterExpression >
type GreaterExpression struct {
	key   string
	value float64
}

// Interpret Interpret
func (g GreaterExpression) Interpret(stats map[string]float64) bool {
	v, ok := stats[g.key]
	if !ok {
		return false
	}
	return v > g.value
}

// NewGreaterExpression NewGreaterExpression
func NewGreaterExpression(exp string) (*GreaterExpression, error) {
	data := regexp.MustCompile(`\s+`).Split(strings.TrimSpace(exp), -1)
	if len(data) != 3 || data[1] != ">" {
		return nil, fmt.Errorf("exp is invalid: %s", exp)
	}

	val, err := strconv.ParseFloat(data[2], 10)
	if err != nil {
		return nil, fmt.Errorf("exp is invalid: %s", exp)
	}

	return &GreaterExpression{
		key:   data[0],
		value: val,
	}, nil
}

// LessExpression <
type LessExpression struct {
	key   string
	value float64
}

// Interpret Interpret
func (g LessExpression) Interpret(stats map[string]float64) bool {
	v, ok := stats[g.key]
	if !ok {
		return false
	}
	return v < g.value
}

// NewLessExpression NewLessExpression
func NewLessExpression(exp string) (*LessExpression, error) {
	data := regexp.MustCompile(`\s+`).Split(strings.TrimSpace(exp), -1)
	if len(data) != 3 || data[1] != "<" {
		return nil, fmt.Errorf("exp is invalid: %s", exp)
	}

	val, err := strconv.ParseFloat(data[2], 10)
	if err != nil {
		return nil, fmt.Errorf("exp is invalid: %s", exp)
	}

	return &LessExpression{
		key:   data[0],
		value: val,
	}, nil
}

// AndExpression &&
type AndExpression struct {
	expressions []IExpression
}

// Interpret Interpret
func (e AndExpression) Interpret(stats map[string]float64) bool {
	for _, expression := range e.expressions {
		if !expression.Interpret(stats) {
			return false
		}
	}
	return true
}

// NewAndExpression NewAndExpression
func NewAndExpression(exp string) (*AndExpression, error) {
	exps := strings.Split(exp, "&&")
	expressions := make([]IExpression, len(exps))

	for i, e := range exps {
		var expression IExpression
		var err error

		switch {
		case strings.Contains(e, ">"):
			expression, err = NewGreaterExpression(e)
		case strings.Contains(e, "<"):
			expression, err = NewLessExpression(e)
		default:
			err = fmt.Errorf("exp is invalid: %s", exp)
		}

		if err != nil {
			return nil, err
		}

		expressions[i] = expression
	}

	return &AndExpression{expressions: expressions}, nil
}

```



### 单元测试

```go
package interpreter

import (
	"testing"

	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

func TestAlertRule_Interpret(t *testing.T) {
	stats := map[string]float64{
		"a": 1,
		"b": 2,
		"c": 3,
	}
	tests := []struct {
		name  string
		stats map[string]float64
		rule  string
		want  bool
	}{
		{
			name:  "case1",
			stats: stats,
			rule:  "a > 1 && b > 10 && c < 5",
			want:  false,
		},
		{
			name:  "case2",
			stats: stats,
			rule:  "a < 2 && b > 10 && c < 5",
			want:  false,
		},
		{
			name:  "case3",
			stats: stats,
			rule:  "a < 5 && b > 1 && c < 10",
			want:  true,
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			r, err := NewAlertRule(tt.rule)
			require.NoError(t, err)
			assert.Equal(t, tt.want, r.Interpret(tt.stats))
		})
	}
}

```