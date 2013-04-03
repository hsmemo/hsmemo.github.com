---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/jfieldIDWorkaround.hpp

### 名前(function name)
```
  static jfieldID to_static_jfieldID(JNIid* id) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) id 引数で渡されたポインタ値を (jfieldID 型にキャストして) リターンするだけ.
      ---------------------------------------- -}

	    assert(id->is_static_field_id(), "from_JNIid, but not static field id");
	    jfieldID result = (jfieldID) id;
	    assert(from_static_jfieldID(result) == id, "must produce the same static id");
	    return result;
	  }
	
```


