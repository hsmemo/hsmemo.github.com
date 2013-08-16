---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.inline.hpp
### 説明(description)

```
// macros for interruptible io and system calls and system call restarting

```

### 名前(function name)
```
#define _INTERRUPTIBLE(_setup, _cmd, _result, _thread, _clear, _before, _after, _int_enable) \
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) まず _setup と _before を実行する.
    
      次に以下のどれかを行う.
      * _int_enable が false だったり, _thread->has_last_Java_frame() が false なら
        単に _cmd を実行する.
      * そうではなく, もしこの時点で os::interupted() が true だったら
        _result に OS_INTRPT をセットするだけ.
      * 上記のいずれでもなければ 
        _cmd を実行する. 
        もしエラーが起こった場合には (返値が負値 && errno が EINTR && os::interupted() が true の場合には), 
        _result に OS_INTRPT をセットする.
    
      最後に _after を実行して終了.
      ---------------------------------------- -}

	do { \
	  _setup; \
	  _before; \
	  OSThread* _osthread = _thread->osthread(); \
	  if (_int_enable && _thread->has_last_Java_frame()) { \
	    /* this is java interruptible io stuff */ \
	    if (os::is_interrupted(_thread, _clear))  { \
	      os::Solaris::bump_interrupted_before_count(); \
	      _result = OS_INTRPT; \
	    } else { \
	      /* _cmd always expands to an assignment to _result */ \
	      if ((_cmd) < 0 && errno == EINTR  \
	       && os::is_interrupted(_thread, _clear)) { \
	        os::Solaris::bump_interrupted_during_count(); \
	        _result = OS_INTRPT; \
	      } \
	    } \
	  } else { \
	    /* this is normal blocking io stuff */ \
	    _cmd; \
	  } \
	  _after; \
	} while(false)
	
```


