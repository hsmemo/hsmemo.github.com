---
layout: default
title: Thread の強制終了処理 (java.lang.Thread.stop() の処理) 
---
[Up](no1IkYYOWe.html) [Top](../index.html)

#### Thread の強制終了処理 (java.lang.Thread.stop() の処理) 

--- 
## 概要(Summary)
処理対象が自スレッドの場合には, THROW_OOP() マクロで例外を投げるだけ.

処理対象が他スレッドの場合には, VM_ThreadStop で VM Operation を発生させ, そのスレッド内に pending exception として埋め込む
(See: VM_ThreadStop).

## 備考(Notes)
このメソッドは deprecated.

## 処理の流れ (概要)(Execution Flows : Summary)
```
java.lang.Thread.stop()
-> java.lang.Thread.stop(Throwable obj)
   -> JVM_StopThread() (= java.lang.Thread.stop0())
      -> 処理対象のスレッドに応じて, 以下のどれかを呼び出す.
         * 処理対象がいない場合 (まだ開始していない or もう死んでいる)
           -> java_lang_Thread::set_stillborn()  (まだ開始していない場合)
         * 処理対象が自分自身の場合
           -> THROW_OOP() マクロ
         * 処理対象が自分以外の場合
           -> Thread::send_async_exception()
              -> VMThread::execute()
                 -> (See: [here](no2935qaz.html) for details)
                    -> VM_ThreadStop::doit()
                       -> JavaThread::send_thread_stop()
                          -> JavaThread::set_pending_async_exception()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.Thread.stop()
See: [here](no2114laq.html) for details
### java.lang.Thread.stop(Throwable obj)
See: [here](no2114ykw.html) for details

### JVM_StopThread()
See: [here](no2114_u2.html) for details
#### 備考(Notes)
java.lang.Thread.stop0() は JVM_StopThread() で実装されている.


```
    ((cite: jdk/src/share/native/java/lang/Thread.c))
    static JNINativeMethod methods[] = {
    ...
        {"stop0",            "(" OBJ ")V", (void *)&JVM_StopThread},
```


### Thread::send_async_exception()
See: [here](no2114LNS.html) for details
### VM_ThreadStop::doit()
See: [here](no2114YXY.html) for details
### JavaThread::send_thread_stop()
See: [here](no2114lhe.html) for details
### JavaThread::set_pending_async_exception()
(#Under Construction)
See: [here](no2114yrk.html) for details






