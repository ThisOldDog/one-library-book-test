[TOC]

---

# 初识ANTLR

> ANTLR（`ANother Tool for Language Recognition`）是一个强大的解析器生成器，用于读取、处理、执行或翻译结构化文本或二进制文件。 ANTLR 根据语法定义生成解析器，解析器可以构建和遍历解析树。

## 概念介绍

- 语言（`Language`）由一系列有意义的语句组成，语句（`Sentence`）由词组组成，词组（`Phrase`）由子词组（`Subphrase`）和词汇符号（`Vocabulary Symbol`）组成。
- 能够分析计算或“执行”语句的程序叫做解释器（`Interpreter`）。
- 能够讲一门语言的语句转换成另一门语言的语句叫做翻译器（`Translator`）。
- 识别语言的程序称为语法分析器（`Parser`）或者句法分析器（`Syntax Analyzer`），句法（`Syntax`）是指约束语言中的各个组成部门之间的规则。语法（`Grammar`）是一些列句法规则的集合，每条规则表述出一种词汇结构。
- **字符聚集为单词或者符号（词法符号，`Token`）的过程称为词法分析（`Lexical Analysis`）或者词法符号化（`Tokenizing`）。文本转换为词法符号的程序称为词法分析器（`Lexer`）。** 词法分析器至少包含两部分信息：词法符号的类型和词法符号文本。
- 语法分析树（`Parse Tree`）包含了分析器识别语句的过程以及语句的各组成部分。语法分析树的内部节点是词组，语法分析树的叶子节点永远是词法符号。

## 使用 ANTLR4 工具快速使用

要使用 ANTLR4 工具的要求是 Python3，通过下面的命令创建`anltr4`和`antlr4-parse`可执行文件：

```bash
$ pip install antlr4-tools
$ antlr4 
Downloading antlr4-4.13.1-complete.jar
ANTLR tool needs Java to run; install Java JRE 11 yes/no (default yes)? y
Installed Java in /Users/parrt/.jre/jdk-11.0.15+10-jre; remove that dir to uninstall
ANTLR Parser Generator  Version 4.13.1
 -o ___              specify output directory where all output is generated
 -lib ___            specify location of grammars, tokens files
...
```

让我们尝试编写一个简单的语法文件 `Expr.g4`：

```
grammar Expr;		
prog:	expr EOF ;
expr:	expr ('*'|'/') expr
    |	expr ('+'|'-') expr
    |	INT
    |	'(' expr ')'
    ;
NEWLINE : [\r\n]+ -> skip;
INT     : [0-9]+ ;
```

### 尝试使用示例语法进行分析

要解析和获取文本形式的解析树，请使用：

```bash
$ antlr4-parse Expr.g4 prog -tree
10+20*30
^D
(prog:1 (expr:2 (expr:3 10) + (expr:1 (expr:3 20) * (expr:3 30))) <EOF>)
```
**注意**：`^D`表示`control-D`，在Unix上表示“输入结束”，在Windows上使用`^Z`。

下面介绍如何通过解析获取词法符号和追朔：

```bash
$ antlr4-parse Expr.g4 prog -tokens -trace
10+20*30
^D
[@0,0:1='10',<INT>,1:0]
[@1,2:2='+',<'+'>,1:2]
[@2,3:4='20',<INT>,1:3]
[@3,5:5='*',<'*'>,1:5]
[@4,6:7='30',<INT>,1:6]
[@5,9:8='<EOF>',<EOF>,2:0]
enter   prog, LT(1)=10
enter   expr, LT(1)=10
consume [@0,0:1='10',<8>,1:0] rule expr
enter   expr, LT(1)=+
consume [@1,2:2='+',<3>,1:2] rule expr
enter   expr, LT(1)=20
consume [@2,3:4='20',<8>,1:3] rule expr
enter   expr, LT(1)=*
consume [@3,5:5='*',<1>,1:5] rule expr
enter   expr, LT(1)=30
consume [@4,6:7='30',<8>,1:6] rule expr
exit    expr, LT(1)=<EOF>
exit    expr, LT(1)=<EOF>
exit    expr, LT(1)=<EOF>
consume [@5,9:8='<EOF>',<-1>,2:0] rule prog
exit    prog, LT(1)=<EOF>
```

下面介绍如何获取可视化树视图：

```bash
$ antlr4-parse Expr.g4 prog -gui
10+20*30
^D
```

以下内容将在基于 Java 的 GUI 窗口中弹出：

![初识ANTLR-20231007020625341.png](./%E5%88%9D%E8%AF%86ANTLR-20231007020625341.png)

### 生成解析器代码

上一节使用了内置的 ANTLR 解释器，但通常您会要求 ANTLR 以项目使用的语言生成代码（从 4.11 开始，大约有 10 种语言可供选择）。 以下是从语法生成 Java 代码的方法：

```bash
$ antlr4 Expr.g4
$ ls Expr*.java
ExprBaseListener.java  ExprLexer.java         ExprListener.java      ExprParser.java
```

下面介绍如何从相同的语法生成C++代码：

```bash
$ antlr4 -Dlanguage=Cpp Expr.g4
$ ls Expr*.cpp Expr*.h
ExprBaseListener.cpp  ExprLexer.cpp         ExprListener.cpp      ExprParser.cpp
ExprBaseListener.h    ExprLexer.h           ExprListener.h        ExprParser.h
```