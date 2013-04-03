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
bool VM_GetOrSetLocal::doit_prologue() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) VM_GetOrSetLocal::get_java_vframe() を呼んで, 
      指定されたフレームに対応する javaVFrame を取得する.
  
      (結果が NULL だったら, ここで false をリターン)
      ---------------------------------------- -}

	  _jvf = get_java_vframe();
	  NULL_CHECK(_jvf, false);
	
  {- -------------------------------------------
  (1) 以下の場合には false をリターンする. それ以外のケースでは true をリターンする.
    
      * 対象のフレームが native method のフレームであり, かつ以下のどちらかが成り立つ場合: (JVMTI_ERROR_OPAQUE_FRAME)
        * 取得対象が receiver ではない
        * (取得対象は receiver だが) そのフレームが static メソッドのフレーム
    
      * 指定された型が対象の局所変数スロットの型と合わない場合:
        (= VM_GetOrSetLocal::check_slot_type() が false をリターンする場合)
      ---------------------------------------- -}

	  if (_jvf->method()->is_native()) {
	    if (getting_receiver() && !_jvf->method()->is_static()) {
	      return true;
	    } else {
	      _result = JVMTI_ERROR_OPAQUE_FRAME;
	      return false;
	    }
	  }
	
	  if (!check_slot_type(_jvf)) {
	    return false;
	  }
	  return true;
	}
	
```


