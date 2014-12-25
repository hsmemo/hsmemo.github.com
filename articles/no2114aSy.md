---
layout: default
title: Thread をスリープさせる処理 (java.lang.Thread.sleep() の処理)  
---
[Up](no1IkYYOWe.html) [Top](../index.html)

#### Thread をスリープさせる処理 (java.lang.Thread.sleep() の処理)  

--- 
## 概要(Summary)
sleep 用のシステムコールを呼び出すだけ.
ただし, 条件によっては yield 用のシステムコールが呼ばれることもある.

なお, os::sleep() は引数の interruptible の値によって処理が異なることに注意.

## 備考(Notes)
なお, java.lang.Thread.sleep() はオーバーロードされており, 2種類存在する.

* java.lang.Thread.sleep(long millis) 
  
  native メソッド. 実体は JVM_Sleep()

* java.lang.Thread.sleep(long millis, int nanos)
  
  native メソッドではない. 
  内部的には java.lang.Thread.sleep(long millis) を呼び出すだけ.
  
  なお, 引数の nanos は「500000 以上なら切り上げて millis を 1つ増加させる」という効果しかない
  (つまり, 今のところ sleep 時間に付いてはミリ秒単位でしか指定はできない).
  といっても, 普通の OS ではミリ秒以下のスリープ要求は聞いてくれないだろうが...
  (<= いや自分でビジーループすればいいのか. まぁデスケジューリングされたら終わりだけど...)


## 処理の流れ (概要)(Execution Flows : Summary)
```
JVM_Sleep() (= java.lang.Thread.sleep(long millis))
-> (条件に応じて, 以下のどれかを呼び出す)
   * 引数の sleep 時間が 0 で, ConvertSleepToYield オプションも指定されている
     -> os::yield()
        -> OS によって処理が異なる.
           * Linux の場合
             -> sched_yield()  (<= スレッドが SCHED_OTHER なんだが, この場合も sched_yield() でいいんだっけ?? 要確認 #TODO)
           * Solaris の場合
             -> os::sleep()    (<= 引数の interruptible は false)
                -> (上述)
           * Windows の場合
             -> os::NakedYield()
                以下のどちらかを呼び出す.
                -> SwitchToThread()
                -> Sleep()
   * 引数の sleep 時間が 0 で, ConvertSleepToYield オプションは指定されていない
     -> os::sleep()    (<= 引数の interruptible は false)
        -> OS によって処理が異なる.
           * Linux の場合
             -> ParkEvent::park()
           * Solaris の場合
             以下のどちらかを呼び出す
             -> thr_yield()  (<= 指定されたスリープ時間が 0 以下の場合)
             -> os_sleep()   (<= それ以外の場合)
           * Windows の場合
             -> Sleep()
   * 引数の sleep 時間が 0 ではない
     -> os::sleep()    (<= 引数の interruptible は true)
        * Linux の場合
          -> os::is_interrupted()
          -> ParkEvent::park()
          -> JavaThread::check_and_wait_while_suspended()   (See: [here](no2114zBI.html) for details)
             -> JavaThread::handle_special_suspend_equivalent_condition()
             -> JavaThread::java_suspend_self()
        * Solaris の場合
          -> os_sleep()
             -> (上述)
          -> JavaThread::check_and_wait_while_suspended()   (See: [here](no2114zBI.html) for details)
             -> (上述)
        * Windows の場合
          -> WaitForMultipleObjects()
          -> JavaThread::check_and_wait_while_suspended()   (See: [here](no2114zBI.html) for details)
             -> (上述)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### JVM_Sleep()
See: [here](no2114YlA.html) for details
#### 備考(Notes)
java.lang.Thread.sleep(long millis) は JVM_Sleep() で実装されている.


```cpp
    ((cite: jdk/src/share/native/java/lang/Thread.c))
    static JNINativeMethod methods[] = {
    ...
        {"sleep",            "(J)V",       (void *)&JVM_Sleep},
```


### os::yield() (Linux の場合)
See: [here](no2114LUG.html) for details
### os::yield() (Solaris の場合)
See: [here](no2114YeM.html) for details
### os::yield() (Windows の場合)
See: [here](no2114MHl.html) for details
### os::NakedYield()  (Windows の場合)
See: [here](no2114ZRr.html) for details

### os::sleep() (Linux の場合)
(#Under Construction)
See: [here](no2114MAx.html) for details
### os::javaTimeNanos() (Linux の場合)
See: [here](no2114lvG.html) for details
### os::sleep() (Solaris の場合)
(#Under Construction)
See: [here](no2114yyY.html) for details
### os_sleep() (Solaris の場合)
See: [here](no2114NWU.html) for details
### INTERRUPTIBLE_NORESTART_VM_ALWAYS() (Solaris の場合)
See: [here](no2114NdI.html) for details
### _INTERRUPTIBLE() (Solaris の場合)
See: [here](no2114anO.html) for details

### os::sleep() (Windows の場合)
(#Under Construction)
See: [here](no2114mbx.html) for details

### JavaThread::check_and_wait_while_suspended()
See: [here](no2114y5M.html) for details
### JavaThread::handle_special_suspend_equivalent_condition()
See: [here](no21140ty.html) for details
### JavaThread::is_external_suspend()
See: [here](no2114AMO.html) for details
### JavaThread::java_suspend_self()
(#Under Construction)
See: [here](no2114m3B.html) for details






