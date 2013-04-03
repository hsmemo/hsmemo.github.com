---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp
### 説明(description)

```
// append a copy of the element to the end of the collection
```

### 名前(function name)
```
void GrowableCache::append(GrowableElement* e) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) GrowableElement::clone() (をサブクラスがオーバーライドしたもの) を呼び出して
      e 引数のコピーを作成し, 
      それを GrowableArray::append() で _elements フィールド内に格納する.
      ---------------------------------------- -}

	  GrowableElement *new_e = e->clone();
	  _elements->append(new_e);

  {- -------------------------------------------
  (1) GrowableCache::recache() を呼んで _cache フィールドの値を更新しておく.
      ---------------------------------------- -}

	  recache();
	}
	
```


