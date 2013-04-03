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
void JvmtiBreakpoints::clearall_at_safepoint() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SafepointSynchronize::is_at_safepoint(), "must be at safepoint");
	
  {- -------------------------------------------
  (1) _bps 内に記録されている全てのブレークポイント箇所に対して JvmtiBreakpoint::clear() を呼び出し, 
      それらをブレークポイントから元に戻す.
      ---------------------------------------- -}

	  int len = _bps.length();
	  for (int i=0; i<len; i++) {
	    _bps.at(i).clear();
	  }

  {- -------------------------------------------
  (1) JvmtiBreakpointCache::clear() を呼んで, _bps 内から要素を全て削除する.
      ---------------------------------------- -}

	  _bps.clear();
	}
	
```


