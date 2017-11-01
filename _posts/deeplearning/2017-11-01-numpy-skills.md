---
layout: post
title: Numpy笔记
category: 深度学习
tags: 深度学习
keywords: Numpy
---

## 库管理

### 生成m行n列零矩阵

```python
temp = np.zeros((m, n)) # 主要m,n要单独括起来
```

### 矩阵A和B对应元素相乘，直接用*号

```python
A = np.zeros((2,3))
B = np.zeros((2,3))
temp = A*B # 直接用*号
```