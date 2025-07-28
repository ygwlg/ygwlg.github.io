---
title: python集合操作和内建函数
date: 2025-05-17 15:29:03
tags:
---

```python
>>> a = {1,2,3}
>>> b = {2,3,4}
```

## 集合包含

```python
a.issubset(b)
a <= b
```

## 并集

```python
a.union(b)
a | b
```

## 交集

```python
a.intersection(b)
a & b
```

## 差集

```python
a.difference(b)
a - b
```

## 对称差分

```python
a.symmetric_difference(b)
a ^ b
```

## 集合更新

```python
a.update(b)
a |= b
```