---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/utilities/exceptions.cpp
### 説明(description)

```
// Convenience method. Calls either the <init>() or <init>(String) method when
// creating a new exception
```

### 名前(function name)
```
Handle Exceptions::new_exception(Thread* thread, Symbol* h_name,
                                 const char* message, Handle h_cause,
                                 Handle h_loader,
                                 Handle h_protection_domain,
                                 ExceptionMsgToUtf8Mode to_utf8_safe) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaCallArguments args;
	  Symbol* signature = NULL;

  {- -------------------------------------------
  (1) 引数で指定されたメッセージ(message)を格納するための java.lang.String オブジェクトを生成する.
      (なお, message が NULL なら何もしない)
  
      (なお, この段階で例外が起きていたり String の確保処理で例外が起きた場合は, 
       処理はその段階で打ち切り, 発生した例外を返値として返す)
      ---------------------------------------- -}

	  if (message == NULL) {
	    signature = vmSymbols::void_method_signature();
	  } else {
	    // We want to allocate storage, but we can't do that if there's
	    // a pending exception, so we preserve any pending exception
	    // around the allocation.
	    // If we get an exception from the allocation, prefer that to
	    // the exception we are trying to build, or the pending exception.
	    // This is sort of like what PRESERVE_EXCEPTION_MARK does, except
	    // for the preferencing and the early returns.
	    Handle incoming_exception (thread, NULL);
	    if (thread->has_pending_exception()) {
	      incoming_exception = Handle(thread, thread->pending_exception());
	      thread->clear_pending_exception();
	    }
	    Handle msg;
	    if (to_utf8_safe == safe_to_utf8) {
	      // Make a java UTF8 string.
	      msg = java_lang_String::create_from_str(message, thread);
	    } else {
	      // Make a java string keeping the encoding scheme of the original string.
	      msg = java_lang_String::create_from_platform_dependent_str(message, thread);
	    }
	    if (thread->has_pending_exception()) {
	      Handle exception(thread, thread->pending_exception());
	      thread->clear_pending_exception();
	      return exception;
	    }
	    if (incoming_exception.not_null()) {
	      return incoming_exception;
	    }
	    args.push_oop(msg);
	    signature = vmSymbols::string_void_signature();
	  }

  {- -------------------------------------------
  (1) Exceptions::new_exception() を呼んで, 例外オブジェクトの生成と初期化を行う.
      ---------------------------------------- -}

	  return new_exception(thread, h_name, signature, &args, h_cause, h_loader, h_protection_domain);
	}
	
```


