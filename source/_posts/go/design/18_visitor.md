---
title: 访问者模式
abbrlink: go/design/visitor
date: 2021-04-13 19:34:02
tags: [go]
categories: [go]
sticky: 1
---

## 访问者模式
> 定义：允许一个或者多个操作应用到一组对象上，解藕操作和对象本身


<!--more-->


### 代码实现

```go
package visitor

import (
	"fmt"
	"path"
)

// Visitor 访问者
type Visitor interface {
	Visit(IResourceFile) error
}

// IResourceFile IResourceFile
type IResourceFile interface {
	Accept(Visitor) error
}

// NewResourceFile NewResourceFile
func NewResourceFile(filepath string) (IResourceFile, error) {
	switch path.Ext(filepath) {
	case ".ppt":
		return &PPTFile{path: filepath}, nil
	case ".pdf":
		return &PdfFile{path: filepath}, nil
	default:
		return nil, fmt.Errorf("not found file type: %s", filepath)
	}
}

// PdfFile PdfFile
type PdfFile struct {
	path string
}

// Accept Accept
func (f *PdfFile) Accept(visitor Visitor) error {
	return visitor.Visit(f)
}

// PPTFile PPTFile
type PPTFile struct {
	path string
}

// Accept Accept
func (f *PPTFile) Accept(visitor Visitor) error {
	return visitor.Visit(f)
}

// Compressor 实现压缩功能
type Compressor struct{}

// Visit 实现访问者模式方法
// 我们可以发现由于没有函数重载，我们只能通过断言来根据不同的类型调用不同函数
// 但是我们即使不采用访问者模式，我们其实也是可以这么操作的
func (c *Compressor) Visit(r IResourceFile) error {
	switch f := r.(type) {
	case *PPTFile:
		return c.VisitPPTFile(f)
	case *PdfFile:
		return c.VisitPDFFile(f)
	default:
		return fmt.Errorf("not found resource typr: %#v", r)
	}
}

// VisitPPTFile VisitPPTFile
func (c *Compressor) VisitPPTFile(f *PPTFile) error {
	fmt.Println("this is ppt file")
	return nil
}

// VisitPDFFile VisitPDFFile
func (c *Compressor) VisitPDFFile(f *PdfFile) error {
	fmt.Println("this is pdf file")
	return nil
}

```



### 单元测试

```go
package visitor

import (
	"testing"

	"github.com/stretchr/testify/require"
)

func TestCompressor_Visit(t *testing.T) {
	tests := []struct {
		name    string
		path    string
		wantErr string
	}{
		{
			name: "pdf",
			path: "./xx.pdf",
		},
		{
			name: "ppt",
			path: "./xx.ppt",
		},
		{
			name:    "404",
			path:    "./xx.xx",
			wantErr: "not found file type",
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			f, err := NewResourceFile(tt.path)
			if tt.wantErr != "" {
				require.Error(t, err)
				require.Contains(t, err.Error(), tt.wantErr)
				return
			}

			require.NoError(t, err)
			compressor := &Compressor{}
			f.Accept(compressor)
		})
	}
}

// 不用 Accept 其实也是可以的
func TestCompressor_Visit2(t *testing.T) {
	tests := []struct {
		name    string
		path    string
		wantErr string
	}{
		{
			name: "pdf",
			path: "./xx.pdf",
		},
		{
			name: "ppt",
			path: "./xx.ppt",
		},
		{
			name:    "404",
			path:    "./xx.xx",
			wantErr: "not found file type",
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			f, err := NewResourceFile(tt.path)
			if tt.wantErr != "" {
				require.Error(t, err)
				require.Contains(t, err.Error(), tt.wantErr)
				return
			}

			require.NoError(t, err)
			compressor := &Compressor{}
			compressor.Visit(f)
		})
	}
}

```