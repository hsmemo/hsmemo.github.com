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
// Constructor for object getter
```

### 名前(function name)
```
VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, JavaThread* calling_thread, jint depth, int index)
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  : _thread(thread)
	  , _calling_thread(calling_thread)
	  , _depth(depth)
	  , _index(index)
	  , _type(T_OBJECT)
	  , _set(false)
	  , _jvf(NULL)
	  , _result(JVMTI_ERROR_NONE)
	{
	}
	
```


