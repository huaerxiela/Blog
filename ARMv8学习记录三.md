---
title: ARMv8学习记录三
date: 2020-03-16 21:10:15
tags: 汇编
category: ARMv8汇编
---

# 除法
##  除以 2^n
算法:
```
if(x >= 0){
    x >> n;
}else{
    (x + 2^n -1) >> n;
}
```

例子：

double i = argc / 4;
```
ADD             W8, W0, #3          
CMP             W0, #0
CSEL            W8, W8, W0, LT
ASR             W1, W8, #2
```