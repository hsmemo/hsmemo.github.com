---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiRedefineClasses.cpp

### 名前(function name)
```
bool VM_RedefineClasses::doit_prologue() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コンストラクタ引数が不正な値であれば, ここで false をリターン.
      ---------------------------------------- -}

	  if (_class_count == 0) {
	    _res = JVMTI_ERROR_NONE;
	    return false;
	  }
	  if (_class_defs == NULL) {
	    _res = JVMTI_ERROR_NULL_POINTER;
	    return false;
	  }
	  for (int i = 0; i < _class_count; i++) {
	    if (_class_defs[i].klass == NULL) {
	      _res = JVMTI_ERROR_INVALID_CLASS;
	      return false;
	    }
	    if (_class_defs[i].class_byte_count == 0) {
	      _res = JVMTI_ERROR_INVALID_CLASS_FORMAT;
	      return false;
	    }
	    if (_class_defs[i].class_bytes == NULL) {
	      _res = JVMTI_ERROR_NULL_POINTER;
	      return false;
	    }
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  // Start timer after all the sanity checks; not quite accurate, but
	  // better than adding a bunch of stop() calls.
	  RC_TIMER_START(_timer_vm_op_prologue);
	
  {- -------------------------------------------
  (1) VM_RedefineClasses::load_new_class_versions() を呼んで, 新しいクラス定義情報を読み込む.
      (なお, もし失敗した場合はここで false をリターン)
      ---------------------------------------- -}

	  // We first load new class versions in the prologue, because somewhere down the
	  // call chain it is required that the current thread is a Java thread.
	  _res = load_new_class_versions(Thread::current());
	  if (_res != JVMTI_ERROR_NONE) {
	    // Free os::malloc allocated memory in load_new_class_version.
	    os::free(_scratch_classes);
	    RC_TIMER_STOP(_timer_vm_op_prologue);
	    return false;
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  RC_TIMER_STOP(_timer_vm_op_prologue);

  {- -------------------------------------------
  (1) true をリターン
      ---------------------------------------- -}

	  return true;
	}
	
```


