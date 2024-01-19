---
title: 享元模式
abbrlink: go/design/flyweight
date: 2021-04-04 18:18:38
tags: [go]
categories: [go]
sticky: 1
---

## 享元模式

> 享元模式的意图是复用对象，节省内存，前提是享元对象是不可变对象。
<!--more-->
> 享元：对象级别的复用；也可以是对象的字段进行复用
> 实现方式：主要通过工厂模式，在工厂类中，通过一个Map来缓存已经创建过的享元对象，来达到复用的目的

### 代码实现

```go
package design

// ChessPieceUnit 棋子享元
type ChessPieceUnit struct {
    ID    uint
    Name  string
    Color string
}

var units = map[int]*ChessPieceUnit{
    1: {
        ID:    1,
        Name:  "車",
        Color: "red",
    },
    2: {
        ID:    2,
        Name:  "炮",
        Color: "red",
    },
    // ... 其他棋子
}

// NewChessPieceUnit 工厂
func NewChessPieceUnit(id int) *ChessPieceUnit {
    return units[id]
}

// ChessPiece 棋子
type ChessPiece struct {
    Unit *ChessPieceUnit
    X    int
    Y    int
}


// ChessBoard 棋局
type ChessBoard struct {
     chessPieces map[int]*ChessPiece
}



// NewChessBoard 初始化棋盘
func NewChessBoard() *ChessBoard {
    board := &ChessBoard{chessPieces: map[int]*ChessPiece{}}
    for id := range units {
        board.chessPieces[id] = &ChessPiece{
            Unit: NewChessPieceUnit(id),
            X:    0,
            Y:    0,
        }
    }
    return board
}

// Move 移动棋子
func (c *ChessBoard) Move(id, x, y int) {
    c.chessPieces[id].X = x
    c.chessPieces[id].Y = y
}
```

### 单元测试

```go
package design

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestNewChessBoard(t *testing.T) {
    board1 := NewChessBoard()
    board1.Move(1, 1, 2)
    board2 := NewChessBoard()
    board2.Move(2, 2, 3)

    assert.Equal(t, board1.chessPieces[1].Unit, board2.chessPieces[1].Unit)
    assert.Equal(t, board1.chessPieces[2].Unit, board2.chessPieces[2].Unit)
}
```
