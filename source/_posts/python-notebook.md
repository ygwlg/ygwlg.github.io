---
title: Python 文档笔记
date: 2025-02-17 09:17:27
tags: 
   - Python
categories: Python
---

## 控制流语句

### match语句

~~早就想知道Python有没有类似switch-case的语法，问了一万个AI都说没有，合着就是Ctrl-F switch是吧。。。~~

#### 1. 拷贝实例代码运行

pycharm说语法错误，原来是3.10引入的新特性；换个3.11的cpython再试试

```python
def http_error(status):
    match status:
        case 400:
            return "Bad request"
        case 404:
            return "Not found"
        case 418:
            return "I'm a teapot"
        case _:
            return "Something's wrong with the internet"
```
#### 2. case _ 

其中 "_"是默认值，那么是py310同时引入了一个关键字"_"吗？直接打印试试

```python
print(_) 

ERROR:....
```

上述代码报错了，由此可见 case _ 是一种特殊的语法


#### 3. case a | b | c

初步猜测是解释成case a: ... case b: ... case c: ... ；不然先计算a|b|c再作为case的参数那就没法匹配a,b,c了


#### 4. cpython语法定义

```grammar
# cpython/Grammar/python.gram
match_stmt[stmt_ty]:
    | "match" subject=subject_expr ':' NEWLINE INDENT cases[asdl_match_case_seq*]=case_block+ DEDENT {
        CHECK_VERSION(stmt_ty, 10, "Pattern matching is", _PyAST_Match(subject, cases, EXTRA)) }

subject_expr[expr_ty]:
    | value=star_named_expression ',' values=star_named_expressions? {
        _PyAST_Tuple(CHECK(asdl_expr_seq*, _PyPegen_seq_insert_in_front(p, value, values)), Load, EXTRA) }
    | named_expression

case_block[match_case_ty]:
    | invalid_case_block
    | "case" pattern=patterns guard=guard? ':' body=block {
        _PyAST_match_case(pattern, guard, body, p->arena) }

guard[expr_ty]: 'if' guard=named_expression { guard }
```
上述四个语法定义的含义:

1. match 声明语句，包括三个部分
  1. match
  2. 匹配的主体subject
  3. :
  4. 多个case代码块
2. 主体表达式，可以为变量名或者 用*拆包的变量
3. case代码块，可以推导除有效和无效(先不关注)的case代码块，其中有效的代码块包括几个部分：
  1. case
  2. patterns, 可以推导成
    1. as表达式
    2. or表达式，对应于上一节中的 '|'隔开的多个表达式
    3. 开放序列 ','隔开的多个表达式
  3. guard 可选的约束项，如果约束项为True则判断当前case
