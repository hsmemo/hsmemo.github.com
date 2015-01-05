---
layout: default
title: Thread が明示的にCPUを明け渡す処理 (java.lang.Thread.yield() の処理)
---
[Up](no1IkYYOWe.html) [Top](../index.html)

#### Thread が明示的にCPUを明け渡す処理 (java.lang.Thread.yield() の処理)

--- 
## 概要(Summary)
yield 用のシステムコールを呼び出すだけ.

## 備考(Notes)
ただし, ConvertYieldToSleep オプションが指定されている場合は, sleep 用のシステムコールを用いる.
また, yield 用のシステムコールが使えない場合も sleep 用のシステムコールを用いる.


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
JVM_Yield() (= java.lang.Thread.yield())
-&gt; ConvertYieldToSleep オプションの値に応じて, 2通りに分岐
   * ConvertYieldToSleep オプションが指定されている場合:
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
   * ConvertYieldToSleep オプションが指定されていない場合:
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
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### JVM_Yield()
See: [here](no2114_1q.html) for details
#### 備考(Notes)
java.lang.Thread.yield() は JVM_Yield() で実装されている.


```cpp
    ((cite: jdk/src/share/native/java/lang/Thread.c))
    static JNINativeMethod methods[] = {
    ...
        {"yield",            "()V",        (void *)&JVM_Yield},
```

### os::dont_yield() (Linux の場合)
See: [here](no2114-JA.html) for details
### os::dont_yield() (Solaris の場合)
See: [here](no2114loS.html) for details
### os::dont_yield() (Windows の場合)
See: [here](no2114_8e.html) for details

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

### os::yield() (Linux の場合)
See: [here](no2114LUG.html) for details
### os::yield() (Solaris の場合)
See: [here](no2114YeM.html) for details
### os::yield() (Windows の場合)
See: [here](no2114MHl.html) for details
### os::NakedYield()  (Windows の場合)
See: [here](no2114ZRr.html) for details






