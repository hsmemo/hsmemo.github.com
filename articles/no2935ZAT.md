---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiThreadState.cpp
### 説明(description)

```
// Helper routine used in several places
```

### 名前(function name)
```
int JvmtiThreadState::count_frames() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  uint32_t debug_bits = 0;
	#endif

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SafepointSynchronize::is_at_safepoint() ||
	         JvmtiEnv::is_thread_fully_suspended(get_thread(), false, &debug_bits),
	         "at safepoint or must be suspended");
	
  {- -------------------------------------------
  (1) Java フレームがなければ 0 をリターン
      ---------------------------------------- -}

	  if (!get_thread()->has_last_Java_frame()) return 0;  // no Java frames
	
  {- -------------------------------------------
  (1) スタック中の javaVFrame の数を数える.
      ---------------------------------------- -}

	  ResourceMark rm;
	  RegisterMap reg_map(get_thread());
	  javaVFrame *jvf = get_thread()->last_java_vframe(&reg_map);
	  int n = 0;
	  // tty->print_cr("CSD: counting frames on %s ...",
	  //               JvmtiTrace::safe_get_thread_name(get_thread()));
	  while (jvf != NULL) {
	    methodOop method = jvf->method();
	    // tty->print_cr("CSD: frame - method %s.%s - loc %d",
	    //               method->klass_name()->as_C_string(),
	    //               method->name()->as_C_string(),
	    //               jvf->bci() );
	    jvf = jvf->java_sender();
	    n++;
	  }

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  // tty->print_cr("CSD: frame count: %d", n);
	  return n;
	}
	
```


