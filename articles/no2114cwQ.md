---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/mutex.cpp
### 説明(description)
単なる Atomic::cmpxchg_ptr() のラッパー.

コメントによると, Atomic::cmpxchg_ptr() と違って引数が標準的な順序になっている, とのこと.
(Atomic::cmpxchg_ptr() は, Sun のインラインテンプレート機能と整合させるために, 変な順序になっているんだとか.
 ちょっと宗教論争的な感じもするが...)

```
// CASPTR() uses the canonical argument order that dominates in the literature.
// Our internal cmpxchg_ptr() uses a bastardized ordering to accommodate Sun .il templates.

```


### 本体部(body)
```
	#define CASPTR(a,c,s) intptr_t(Atomic::cmpxchg_ptr ((void *)(s),(void *)(a),(void *)(c)))
	
```


