---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnv.cpp
### 説明(description)

```
// class_count - pre-checked to be greater than or equal to 0
// class_definitions - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::RedefineClasses(jint class_count, const jvmtiClassDefinition* class_definitions) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) VM_RedefineClasses を呼び出すだけ.
      ---------------------------------------- -}

	//TODO: add locking
	  VM_RedefineClasses op(class_count, class_definitions, jvmti_class_load_kind_redefine);
	  VMThread::execute(&op);
	  return (op.check_error());
	} /* end RedefineClasses */
	
```


