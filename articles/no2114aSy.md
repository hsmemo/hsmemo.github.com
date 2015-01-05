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
<div class="flow-abst"><pre>
JVM_Sleep() (= java.lang.Thread.sleep(long millis))
-&gt; (条件に応じて, 以下のどれかを呼び出す)
   * 引数の sleep 時間が 0 で, ConvertSleepToYield オプションも指定されている
     -&gt; os::yield()
        -&gt; OS によって処理が異なる.
           * Linux の場合
             -&gt; sched_yield()  (&lt;= スレッドが SCHED_OTHER なんだが, この場合も sched_yield() でいいんだっけ?? 要確認 #TODO)
           * Solaris の場合
             -&gt; os::sleep()    (&lt;= 引数の interruptible は false)
                -&gt; (上述)
           * Windows の場合
             -&gt; os::NakedYield()
                以下のどちらかを呼び出す.
                -&gt; SwitchToThread()
                -&gt; Sleep()
   * 引数の sleep 時間が 0 で, ConvertSleepToYield オプションは指定されていない
     -&gt; os::sleep()    (&lt;= 引数の interruptible は false)
        -&gt; OS によって処理が異なる.
           * Linux の場合
             -&gt; ParkEvent::park()
           * Solaris の場合
             以下のどちらかを呼び出す
             -&gt; thr_yield()  (&lt;= 指定されたスリープ時間が 0 以下の場合)
             -&gt; os_sleep()   (&lt;= それ以外の場合)
           * Windows の場合
             -&gt; Sleep()
   * 引数の sleep 時間が 0 ではない
     -&gt; os::sleep()    (&lt;= 引数の interruptible は true)
        * Linux の場合
          -&gt; os::is_interrupted()
          -&gt; ParkEvent::park()
          -&gt; JavaThread::check_and_wait_while_suspended()   (See: <a href="no2114zBI.html">here</a> for details)
             -&gt; JavaThread::handle_special_suspend_equivalent_condition()
             -&gt; JavaThread::java_suspend_self()
        * Solaris の場合
          -&gt; os_sleep()
             -&gt; (上述)
          -&gt; JavaThread::check_and_wait_while_suspended()   (See: <a href="no2114zBI.html">here</a> for details)
             -&gt; (上述)
        * Windows の場合
          -&gt; WaitForMultipleObjects()
          -&gt; JavaThread::check_and_wait_while_suspended()   (See: <a href="no2114zBI.html">here</a> for details)
             -&gt; (上述)
</pre></div>


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






