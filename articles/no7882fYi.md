---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/safepoint.hpp
### 説明(description)

```
  // VM Thread interface for determining safepoint rate
```

### 名前(function name)
```
  static long last_non_safepoint_interval() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 現在時刻 (= os::javaTimeMillis() の値) から
      _end_of_last_safepoint の値を引いたものをリターン.
    
      (_end_of_last_safepoint は, 最後の Safepoint 処理が終わった時刻が記録されている.
       See: SafepointSynchronize::end())
      ---------------------------------------- -}

	    return os::javaTimeMillis() - _end_of_last_safepoint;
	  }
	
```


