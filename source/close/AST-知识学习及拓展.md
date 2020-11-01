---
title: AST-知识学习及拓展
date: 2020-10-30 10:16:24
tags:
---
> 当我开始学习ast时，我们更多的是想，有是什么书可以看？但是我一直没找到满意的书籍，可能国内这部分的书籍还是较少。个人猜测，偏原理性的书籍，更注重权威性，所以比较少。大家可以通过阅读博客对形式，快速学习。

## 前置知识

对于编译型语言（java），编译分为：`词法分析`->`语法分析`->`语义检查`->`代码优化和字节码生产`。
对于解释型语言（javascript），`词法分析`->`语法分析`->`语法树`->开始解释执行。

从上面的执行过程，可以知道两个点，javascript不会生成字节码，而是语法树。这就是 AST (抽象语法树：Abstract Syntax Tree)，相当于编译型语言的编译器。设想一下，掌握了编译器，等同于掌握了所有代码的生死。

## 了解 Javascript AST 运转过程
用个简单的例子
```javscript
var AST = "I'm good boy";
```

1. ### 词法分析
这个过程也叫扫描(scanner)，将有利于开发者查看的字符流（char stream）转成便于处理的数据标记，记号流(token stream)
这个过程会移除注释和空白付符，因为这些，对于代码运行无意义，只是为了提高可维护及美观。

看到全大写的英文，理解起来比较费劲，为了方便理解加上了中文注释，可能中文语义不太契合，理解万岁。
```
NAME "AST"          // 声明 "AST"
EQUALS              // 赋值
NAME "I'm good boy" // 声明 "I'm good boy"
SEMICOLON           // 结束符，分号
```

2. ### 语法分析
你可以理解成解析器，不同的标记，对应着不同的语法。语法分析，主要是将这些标记转换成对应语法的数据结构。
同时也会对代码进行语法校验，有错误，会抛出。
通过语法分析转换后的ast，并不是百分百对于源码的，这语法分析过程中，会剔除一些不合法的东西，比如不完整的括号。
如果你想获取百分百对应源码的树结构，可以去了解一下CST(具体语法树)，这里就不扩展讲了。
[Ast 查看工具](https://astexplorer.net)

既然AST叫抽象语法树，那么它就是有树的结构，也就有节点。
tip:
- `type` 指明节点的类型
- `start` 记录开始的标记位
- `end` 记录结束的标记位

```
// 根
Program {                         // 根节点，表示整个代码块
  type: Program
  start: 0
  end: 25
  body: [                         // 主体
    VariableDeclaration {           // 变量声明节点
      type: VariableDeclaration
      start: 0
      end:25
      declarations: [               // 包含来变量初始值，是变量声明的主要内容
        VariableDeclaration {           // 变量声明节点
          type: VariableDeclaration
          start: 4
          end:24
          id: Identifier {              // 标识符
            type: "Identifier"
            start: 4
            end: 7
            name: "AST"                 // 值
          }
          init: Literal {               // 字面量，计算机公认的一种固定基础类型值的表示法
            type: "Literal"
            start: 10
            end: 24
            value: "I'm good boy"       // 值
            raw: "\"I'm good boy\""     // 未处理值
          }
        }
      ]
      kind: "var"                   // 变量声明的关键字（我们甚至可以通过遍历语法树，将所有var改为let）
    }
  ]
  sourceType: "module"          // 表示代码类型
}
```

总结：
1. AST 把我们的代码用数据结构来表示，抽象化，特别符合它名字，抽象语法树。（树的结构，通过抽象出代码的通用结构）
2. 抽象的后的代码，更利于我们去些事情。比如，代码风格检查（eslint）、语法高亮、错误提示、格式化代码（prettier）、主动补全、代码混淆压缩（打包优化相关）、改变代码结构(某些跨端框架原理)、兼容（polyfill）等

### 利用工具实际练习
首先，认识一些工具
esprima: 可以将 js 代码转换成 `ast`
estraverse: 遍历 `ast` ，可以执行一些替换、增加、删除等操作
escodegen： 将 `ast` 转换成代码。

