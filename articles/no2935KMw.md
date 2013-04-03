---
layout: default
title: java.lang.Thread オブジェクト (= JavaThread オブジェクト) を生成する側の処理 (2) ： `java.lang.Thread.start()` の処理  
---
[Up](no_adcwNt_.html) [Top](../index.html)

#### java.lang.Thread オブジェクト (= JavaThread オブジェクト) を生成する側の処理 (2) ： `java.lang.Thread.start()` の処理  

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
```
java.lang.Thread.start()
-> java.lang.ThreadGroup.add()
-> java.lang.ThreadGroup.start0()
   -> JVM_StartThread()
      -> (1) 新しい JavaThread オブジェクトを生成する
             -> JavaThread::JavaThread()   (<= なお, エントリポイントとしては thread_entry() 関数が指定されている)
                -> Thread::Thread()
                -> JavaThread::initialize()
                -> JavaThread::set_entry_point()
                -> os::create_thread()
                   -> (See: [here](noYHbL-pQM.html) for details)

         (1) 生成した JavaThread オブジェクトを初期化する
             -> JavaThread::prepare()
                -> JavaThread::set_thread()
                -> java_lang_Thread::set_thread()
                -> Thread::set_priority()
                -> Threads::add()

         (1) 生成した JavaThread の実行を開始させる.
             -> Thread::start()
                -> java_lang_Thread::set_thread_status()
                -> os::start_thread()
                   -> (See: [here](noYHbL-pQM.html) for details)

(なお, java.lang.ThreadGroup.start0() での生成に失敗した場合は以下の関数が呼ばれる)
-> java.lang.ThreadGroup.threadStartFailed()
   -> java.lang.ThreadGroup.remove()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.Thread.start()
See: [here](no2114jYc.html) for details
### java.lang.ThreadGroup.add()
See: [here](no2114wii.html) for details
### java.lang.ThreadGroup.threadStartFailed()
See: [here](no21149so.html) for details
### java.lang.ThreadGroup.remove()
See: [here](no2114XB1.html) for details
### JVM_StartThread()
See: [here](no2114JLE.html) for details
### java_lang_Thread::stackSize()
(#Under Construction)

### JavaThread::JavaThread()
See: [here](no2114jfQ.html) for details
### Thread::Thread()
See: [here](no2114wpW.html) for details
### JavaThread::initialize()
See: [here](no2114XIp.html) for details
### ThreadProfiler::engage()
See: [here](no3059XRI.html) for details
### ThreadSafepointState::create()
See: [here](no3059KHC.html) for details
### JavaThread::pd_initialize()  (Linux x86 の場合)
See: [here](no2114kSv.html) for details
### JavaThread::pd_initialize()  (Solaris sparc の場合)
See: [here](no3059S3I.html) for details
### JavaThread::pd_initialize()  (Windows x86 の場合)
See: [here](no3059fBP.html) for details
### JavaThread::set_entry_point()
See: [here](no21149zc.html) for details
### JavaThread::prepare()
See: [here](no3059fID.html) for details
### JavaThread::set_threadObj()
See: [here](no3059GnV.html) for details
### JavaThread::threadObj()
See: [here](no2114kgX.html) for details
### java_lang_Thread::set_thread()
See: [here](no3059Txb.html) for details
### java_lang_Thread::priority()
(#Under Construction)

### Thread::set_priority()
See: [here](no2114lT2.html) for details
### Threads::add()
See: [here](no3059g7h.html) for details
### JavaThread::initialize_queues()  (#ifndef SERIALGC の場合)
See: [here](no3059kbO.html) for details
### JavaThread::initialize_queues()  (#ifdef SERIALGC の場合)
See: [here](no3059xlU.html) for details
### ThreadService::add_thread()
See: [here](no3059tFo.html) for details
### Thread::start()
See: [here](no2114WVK.html) for details
### java_lang_Thread::set_thread_status()
See: [here](no30596Pu.html) for details






