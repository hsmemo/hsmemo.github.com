---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp

### 名前(function name)
```
void  JvmtiCurrentBreakpoints::listener_fun(void *this_obj, address *cache) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JvmtiBreakpoints *this_jvmti = (JvmtiBreakpoints *) this_obj;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(this_jvmti != NULL, "this_jvmti != NULL");
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  debug_only(int n = this_jvmti->length(););
	  assert(cache[n] == NULL, "cache must be NULL terminated");
	
  {- -------------------------------------------
  (1) JvmtiCurrentBreakpoints::set_breakpoint_list() を呼び出し, 
      JvmtiCurrentBreakpoints::_breakpoint_list フィールドを更新しておく.
      ---------------------------------------- -}

	  set_breakpoint_list(cache);
	}
	
```


