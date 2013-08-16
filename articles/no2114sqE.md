---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.ThreadInfo オブジェクトを取得する処理 (java.lang.management.ThreadMXBean.getThreadInfo() および java.lang.management.ThreadMXBean.dumpAllThreads() の処理)  
---
[Up](noMz-1isvk.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.ThreadInfo オブジェクトを取得する処理 (java.lang.management.ThreadMXBean.getThreadInfo() および java.lang.management.ThreadMXBean.dumpAllThreads() の処理)  

--- 
## 概要(Summary)
(See: JSR-174)

## 処理の流れ (概要)(Execution Flows : Summary)
### sun.management.ThreadImpl.getThreadInfo() の処理
```
sun.management.ThreadImpl.getThreadInfo()
-> sun.management.ThreadImpl.getThreadInfo1()
   -> Java_sun_management_ThreadImpl_getThreadInfo1()
      -> jmm_GetThreadInfo()
         (1) 以下のどちらかの方法で ThreadSnapshot オブジェクトを生成
             * スレッドのスタックダンプは必要ない場合:
               -> ThreadSnapshot::ThreadSnapshot()
             * スレッドのスタックダンプまで必要な場合:
               -> do_thread_dump()
                  -> VMThread::execute()
                     -> (See: [here](no2935qaz.html) for details)
                        -> VM_ThreadDump::doit_prologue()
                        -> VM_ThreadDump::doit()
                           -> ConcurrentLocksDump::dump_at_safepoint()        (「現在ロックしているシンクロナイザの一覧」も取得する場合)
                           -> VM_ThreadDump::snapshot_thread()
                              -> ThreadSnapshot::dump_stack_at_safepoint()
                                 -> ThreadStackTrace::dump_stack_at_safepointr()
                                    -> ThreadStackTrace::add_stack_frame()
                                       -> StackFrameInfo::StackFrameInfo()
                                          -> javaVFrame::locked_monitors()    (「現在ロックしているオブジェクトモニターの一覧」も取得する場合)
                                    -> ObjectSynchronizer::monitors_iterate() (「現在ロックしているオブジェクトモニターの一覧」も取得する場合)
                                       -> InflatedMonitorsClosure::do_monitor()
                        -> VM_ThreadDump::doit_epilogue()
         (2) ThreadSnapshot オブジェクトから java.lang.management.ThreadInfo オブジェクトを生成
             -> Management::create_thread_info_instance()
                -> initialize_ThreadInfo_constructor_arguments()
```

### sun.management.ThreadImpl.getThreadInfo(long[] ids, boolean lockedMonitors, boolean lockedSynchronizers) の処理
```
sun.management.ThreadImpl.getThreadInfo(long[] ids, boolean lockedMonitors, boolean lockedSynchronizers)
-> sun.management.ThreadImpl.dumpThreads0()
   -> Java_sun_management_ThreadImpl_dumpThreads0()
      -> jmm_DumpThreads()
         (1) 対象のスレッドの情報を ThreadSnapshot オブジェクトとして取得.
             * 引数で情報取得対象のスレッドが指定されている場合 (getThreadInfo() から呼び出された場合):
               -> do_thread_dump()
                  -> (同上)
             * そうでない場合 (dumpAllThreads() から呼び出された場合):
               -> VMThread::execute()
                  -> (See: [here](no2935qaz.html) for details)
                     -> VM_ThreadDump::doit_prologue()
                     -> VM_ThreadDump::doit()
                        -> (同上)
                     -> VM_ThreadDump::doit_epilogue()
         (2) 「現在ロックしているオブジェクトモニターの一覧」も取得する場合には, 以下を呼び出す.
             -> StackFrameInfo::locked_monitors()
             -> ThreadStackTrace::jni_locked_monitors()
         (3) 「現在ロックしているシンクロナイザの一覧」も取得する場合には, 以下を呼び出す.
             -> ThreadSnapshot::get_concurrent_locks()
             -> ThreadConcurrentLocks::owned_locks()
         (4) 以上で取得した値から java.lang.management.ThreadInfo オブジェクトを生成する.
             -> Management::create_thread_info_instance()
                -> (同上)
```

### sun.management.ThreadImpl.dumpAllThreads() の処理
```
sun.management.ThreadImpl.dumpAllThreads()
-> sun.management.ThreadImpl.dumpThreads0()
   -> (同上)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### sun.management.ThreadImpl.getThreadInfo(long id)
See: [here](no2114qMm.html) for details
### sun.management.ThreadImpl.getThreadInfo(long id, int maxDepth)
See: [here](no21143Ws.html) for details
### sun.management.ThreadImpl.getThreadInfo(long[] ids)
See: [here](no2114Ehy.html) for details
### sun.management.ThreadImpl.getThreadInfo()
See: [here](no21142qB.html) for details
### Java_sun_management_ThreadImpl_getThreadInfo1()
See: [here](no2114D1H.html) for details
### jmm_GetThreadInfo()
See: [here](no2114Q_N.html) for details
### validate_thread_id_array()
See: [here](no21143rI.html) for details
### validate_thread_info_array()
See: [here](no2114E2O.html) for details
### ThreadSnapshot::ThreadSnapshot()
See: [here](no2114dJU.html) for details
### do_thread_dump()
See: [here](no2114e8y.html) for details

### VM_ThreadDump::doit()
See: [here](no2114QGC.html) for details
### ConcurrentLocksDump::dump_at_safepoint()
See: [here](no2114RAV.html) for details
### HeapInspection::find_instances_at_safepoint()
See: [here](no78827rc.html) for details
### FindInstanceClosure::do_object()
See: [here](no7882I2i.html) for details
### ConcurrentLocksDump::build_map()
See: [here](no2114eKb.html) for details
### ConcurrentLocksDump::add_lock()
See: [here](no2114rUh.html) for details
### ConcurrentLocksDump::thread_concurrent_locks()
See: [here](no21144en.html) for details

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
### InflatedMonitorsClosure::do_monitor()
See: [here](no2114HAY.html) for details

### Management::create_thread_info_instance()
See: [here](no2114Fpt.html) for details
### initialize_ThreadInfo_constructor_arguments()
(#Under Construction)
See: [here](no2114E9C.html) for details

### sun.management.ThreadImpl.getThreadInfo(long[] ids, boolean lockedMonitors, boolean lockedSynchronizers)
See: [here](no21143dg.html) for details
### Java_sun_management_ThreadImpl_dumpThreads0()
See: [here](no2114Eom.html) for details
### jmm_DumpThreads()
See: [here](no2114Rys.html) for details
### StackFrameInfo::locked_monitors()
See: [here](no2114trL.html) for details
### ThreadStackTrace::jni_locked_monitors()
See: [here](no211461R.html) for details
### ThreadSnapshot::get_concurrent_locks()
See: [here](no2114uX2.html) for details
### ThreadConcurrentLocks::owned_locks()
See: [here](no2114ghF.html) for details

### Management::create_thread_info_instance()
See: [here](no2114Szz.html) for details

### sun.management.ThreadImpl.dumpAllThreads()
See: [here](no2114qTa.html) for details






