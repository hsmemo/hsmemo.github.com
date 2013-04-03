---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/jniHandles.cpp
### 説明(description)

```
// Determine if the handle is somewhere in the current thread's stack.
// We easily can't isolate any particular stack frame the handle might
// come from, so we'll check the whole stack.

```

### 名前(function name)
```
bool JNIHandles::is_frame_handle(JavaThread* thr, jobject obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) obj 引数で指定されたオブジェクトのアドレスが
      thr 引数で指定された Thread のスタックフレーム内にあれば true をリターン.
      (逆に, なければ false をリターン)
  
      (より正確には, Thread のスタックフレーム内の base から last_Java_sp の範囲, にあれば true.
       この範囲にない場合や last_Java_sp が設定されていない場合は false をリターン)
  
      (なお, active_handles ではなくスタックフレーム上に確保されるケースとしては
       ダミーフレーム上に確保されたネイティブメソッド用の引数等)
      ---------------------------------------- -}

	  // If there is no java frame, then this must be top level code, such
	  // as the java command executable, in which case, this type of handle
	  // is not permitted.
	  return (thr->has_last_Java_frame() &&
	         (void*)obj < (void*)thr->stack_base() &&
	         (void*)obj >= (void*)thr->last_Java_sp());
	}
	
```


