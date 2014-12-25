---
layout: default
title: Thread のスタックフレームを取得する処理 (java.lang.Thread.getStackTrace(), java.lang.Thread.getAllStackTraces() の処理)  
---
[Up](nokB2etACd.html) [Top](../index.html)

#### Thread のスタックフレームを取得する処理 (java.lang.Thread.getStackTrace(), java.lang.Thread.getAllStackTraces() の処理)  

--- 
## 概要(Summary)
どちらも, private method である java.lang.Thread.dumpThreads() を呼び出して処理を行っている.
これはネイティブメソッドになっており, 最終的には VM_ThreadDump によるダンプ処理が行われる
(See: VM_ThreadDump).


## 処理の流れ (概要)(Execution Flows : Summary)
### java.lang.Thread.getStackTrace() の処理
```
java.lang.Thread.getStackTrace()
-> JVM_DumpThreads() (= java.lang.Thread.dumpThreads())
   -> ThreadService::dump_stack_traces()
      -> VMThread::execute()
         -> (See: [here](no2935qaz.html) for details)
            -> VM_ThreadDump::doit_prologue()
            -> VM_ThreadDump::doit()
               -> VM_ThreadDump::snapshot_thread()
                  -> ThreadSnapshot::dump_stack_at_safepoint()
                     -> ThreadStackTrace::dump_stack_at_safepointr()
                        -> ThreadStackTrace::add_stack_frame()
                           -> StackFrameInfo::StackFrameInfo()
            -> VM_ThreadDump::doit_epilogue()
```

### java.lang.Thread.getAllStackTraces() の処理
```
java.lang.Thread.getAllStackTraces()
-> JVM_DumpThreads() (= java.lang.Thread.dumpThreads())
   -> (同上)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.Thread.getStackTrace()
See: [here](no2114_DT.html) for details
### JVM_DumpThreads()
See: [here](no2114ZYf.html) for details
#### 備考(Notes)
java.lang.Thread.dumpThreads() は JVM_DumpThreads() で実装されている.


```cpp
    ((cite: jdk/src/share/native/java/lang/Thread.c))
    static JNINativeMethod methods[] = {
    ...
        {"dumpThreads",      "([" THD ")[[" STE, (void *)&JVM_DumpThreads},
```

### ThreadService::dump_stack_traces()
See: [here](no2114mil.html) for details
### VM_ThreadDump::doit()
See: [here](no2114QGC.html) for details
### VM_ThreadDump::snapshot_thread()
See: [here](no2114R5g.html) for details
### ThreadSnapshot::dump_stack_at_safepoint()
See: [here](no2114eDn.html) for details
### ThreadStackTrace::dump_stack_at_safepoint()
See: [here](no2114rNt.html) for details
### ThreadStackTrace::add_stack_frame()
See: [here](no21144Xz.html) for details
### StackFrameInfo::StackFrameInfo()
(#Under Construction)
See: [here](no2114qhC.html) for details

### java.lang.Thread.getAllStackTraces()
See: [here](no2114MOZ.html) for details






