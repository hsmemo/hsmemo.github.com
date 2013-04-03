---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： JNI による例外処理(Exceptions)
---
[Up](no7882H_v.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： JNI による例外処理(Exceptions)

--- 
## 該当する JNI 関数
* `Throw`,
* `ThrowNew`,
* `ExceptionOccurred`,
* `ExceptionDescribe`,
* `ExceptionClear`,
* `FatalError`,
* `ExceptionCheck`,

## 概要(Summary)
(#Under Construction)


## 処理の流れ (概要)(Execution Flows : Summary)
### Throw() の処理
```
jni_Throw()
-> THROW_OOP_()
   -> (See: [here](no3059qOR.html) for details)
```

### ThrowNew() の処理
```
jni_ThrowNew()
-> THROW_MSG_LOADER_()
   -> (See: [here](no3059qOR.html) for details)
```

### ExceptionOccurred() の処理
```
jni_ExceptionOccurred()
-> jni_check_async_exceptions()
   -> JavaThread::check_and_handle_async_exceptions()
-> ThreadShadow::pending_exception()
```

### ExceptionDescribe() の処理
```
jni_ExceptionDescribe()
-> JavaCalls::call_virtual()
   -> (See: [here](no3059iJu.html) for details)
      -> java.lang.Throwable.printStackTrace()
```

### ExceptionClear() の処理
```
jni_ExceptionClear()
-> ThreadShadow::clear_pending_exception()
```

### FatalError() の処理
```
jni_FatalError()
-> os::abort()
```

### ExceptionCheck() の処理
```
jni_ExceptionCheck()
-> jni_check_async_exceptions()
   -> JavaThread::check_and_handle_async_exceptions()
-> ThreadShadow::has_pending_exception()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_Throw()
See: [here](no3059OJ0.html) for details

### jni_ThrowNew()
See: [here](no3059ATD.html) for details

### jni_ExceptionOccurred()
See: [here](no3059NdJ.html) for details
### jni_check_async_exceptions()
See: [here](no3059BGi.html) for details
### JavaThread::check_and_handle_async_exceptions()
(#Under Construction)
See: [here](no3059OQo.html) for details

### jni_ExceptionDescribe()
See: [here](no3059anP.html) for details

### jni_ExceptionClear()
See: [here](no3059nxV.html) for details

### jni_FatalError()
See: [here](no305907b.html) for details

### jni_ExceptionCheck()
See: [here](no3059bau.html) for details






