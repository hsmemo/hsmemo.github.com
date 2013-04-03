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
// Constructor for non-object getter
```

### 名前(function name)
```
VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type)
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  : _thread(thread)
	  , _calling_thread(NULL)
	  , _depth(depth)
	  , _index(index)
	  , _type(type)
	  , _set(false)
	  , _jvf(NULL)
	  , _result(JVMTI_ERROR_NONE)
	{
	}
	
```


