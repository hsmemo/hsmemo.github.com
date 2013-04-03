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
void JvmtiBreakpoint::set() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) methodOopDesc::set_breakpoint() を引数として
      JvmtiBreakpoint::each_method_version_do() を呼び出す.
  
      (これにより, コンストラクタで指定されたメソッドの (現在及び過去の) 全ての EMCP バージョンに対して
       methodOopDesc::set_breakpoint() が実行され, 
       コンストラクタ引数で指定された箇所にブレークポイントが埋められる.
       EMCP については RedefineClass() 参照.)
      ---------------------------------------- -}

	  each_method_version_do(&methodOopDesc::set_breakpoint);
	}
	
```


