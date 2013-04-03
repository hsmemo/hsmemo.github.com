---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/mutex.cpp
### 説明(description)


```
// Simplistic low-quality Marsaglia SHIFT-XOR RNG.
// Bijective except for the trailing mask operation.
// Useful for spin loops as the compiler can't optimize it away.
```

### 名前(function name)
```
static inline jint MarsagliaXORV (jint x) {
```

### 本体部(body)
```
	コメントによると, これは Marsaglia 氏による乱数生成アルゴリズムを簡略化したもの.
	コンパイラが最適化によって消すことが難しいので, スピンループ内での使用に適している, とのこと.
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (x == 0) x = 1|os::random() ;
	  x ^= x << 6;
	  x ^= ((unsigned)x) >> 21;
	  x ^= x << 7 ;
	  return x & 0x7FFFFFFF ;
	}
	
```


