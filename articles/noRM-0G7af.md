---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： JMM Interface
---
[Up](nokcNcOF9U.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： JMM Interface

--- 
## 概要(Summary)
Platform MXBean 機能の実現のためには
sun.management パッケージ (See: [here](nokcNcOF9U.html) for details) 下のクラスから 
HotSpot 内のデータにアクセスする必要がある.
そのためのインターフェースとして HotSpot に "JMM Interface" というエントリポイントが用意されている.

この "JMM Interface" (及びそこで使われる構造体/定数など) の宣言は jmm.h に書かれている.


```cpp
    ((cite: hotspot/src/share/vm/services/jmm.h))
    /*
     * This is a private interface used by JDK for JVM monitoring
     * and management.
     *
     * Bump the version number when either of the following happens:
     *
     * 1. There is a change in functions in JmmInterface.
     *
     * 2. There is a change in the contract between VM and Java classes.
     */
```

### 参考(for your information)
 * <http://openjdk.java.net/groups/hotspot/docs/Serviceability.html>

   "File src/share/vm/services/jmm.h defines a Sun private interface that is implemented in HotSpot
   and is used by the monitoring and management code in the J2SE repository.
   jmm.h is copied into the J2SE repository so that monitoring and management native methods can use it to call into HotSpot to extract information.
   jmm.h contains a version number that is used at runtime to verify interface compatibility
   between the Java code and the HotSpot that is being monitored."

 * <http://openjdk.java.net/groups/serviceability/svcjdk.html>

   "Java SE contains a JMX MBean Server called the Platform MBean Server (see java.lang.management.ManagementFactory.getPlatformMBeanServer)
   which can be used to make MBeans available for remote access.
   The Platform MBean Server is started by code in j2se/src/share/classes/sun/management/Agent.java.
   This class is named as the Premain-class and Agent-class in JPLIS agent jre/lib/management-agent.jar
   which allows the agent (and thus the Platform MBean Server) to be started dynamically via the loadAgent method of Dynamic Attach.
   The agent can also be started at JVM startup time by use of the -Dcom.sun.management.jmxremote or
   the -Dcom.sun.management.snmp command line options on the java command.
   
   The classes in java.lang.managment define a set of MBeans that are used to access information from the JVM and the operating system upon which it is running. These MBeans are registered with the Platform MBean Server when it is started.
   
   The platform MBeans are implemented by classes in sun.management.
   These classes contain native methods that call functions defined in jmm.h to extract the required information from the JVM.
   jmm.h is implemented by the Java Virtual Machine."


## 備考(Notes)
jmm.h ファイルは hotspot/ 下と jdk/ 下の両方に存在しているが中身は同じ.
それぞれ hotspot/ 下のソースと jdk/ 下のソースから参照されている.

* hotspot/src/share/vm/services/jmm.h
* jdk/src/share/javavm/export/jmm.h

## 備考(Notes)
JMM interface の関数一覧:

```cpp
    ((cite: hotspot/src/share/vm/services/jmm.h))
    typedef struct jmmInterface_1_ {
      void*        reserved1;
      void*        reserved2;
    
      jint         (JNICALL *GetVersion)             (JNIEnv *env);
    
      jint         (JNICALL *GetOptionalSupport)     (JNIEnv *env,
                                                      jmmOptionalSupport* support_ptr);
    
      /* This is used by JDK 6 and earlier.
       * For JDK 7 and after, use GetInputArgumentArray.
       */
      jobject      (JNICALL *GetInputArguments)      (JNIEnv *env);
    
      jint         (JNICALL *GetThreadInfo)          (JNIEnv *env,
                                                      jlongArray ids,
                                                      jint maxDepth,
                                                      jobjectArray infoArray);
      jobjectArray (JNICALL *GetInputArgumentArray)  (JNIEnv *env);
    
      jobjectArray (JNICALL *GetMemoryPools)         (JNIEnv* env, jobject mgr);
    
      jobjectArray (JNICALL *GetMemoryManagers)      (JNIEnv* env, jobject pool);
    
      jobject      (JNICALL *GetMemoryPoolUsage)     (JNIEnv* env, jobject pool);
      jobject      (JNICALL *GetPeakMemoryPoolUsage) (JNIEnv* env, jobject pool);
    
      void         (JNICALL *GetThreadAllocatedMemory)
                                                     (JNIEnv *env,
                                                      jlongArray ids,
                                                      jlongArray sizeArray);
    
      jobject      (JNICALL *GetMemoryUsage)         (JNIEnv* env, jboolean heap);
    
      jlong        (JNICALL *GetLongAttribute)       (JNIEnv *env, jobject obj, jmmLongAttribute att);
      jboolean     (JNICALL *GetBoolAttribute)       (JNIEnv *env, jmmBoolAttribute att);
      jboolean     (JNICALL *SetBoolAttribute)       (JNIEnv *env, jmmBoolAttribute att, jboolean flag);
    
      jint         (JNICALL *GetLongAttributes)      (JNIEnv *env,
                                                      jobject obj,
                                                      jmmLongAttribute* atts,
                                                      jint count,
                                                      jlong* result);
    
      jobjectArray (JNICALL *FindCircularBlockedThreads) (JNIEnv *env);
    
      // Not used in JDK 6 or JDK 7
      jlong        (JNICALL *GetThreadCpuTime)       (JNIEnv *env, jlong thread_id);
    
      jobjectArray (JNICALL *GetVMGlobalNames)       (JNIEnv *env);
      jint         (JNICALL *GetVMGlobals)           (JNIEnv *env,
                                                      jobjectArray names,
                                                      jmmVMGlobal *globals,
                                                      jint count);
    
      jint         (JNICALL *GetInternalThreadTimes) (JNIEnv *env,
                                                      jobjectArray names,
                                                      jlongArray times);
    
      jboolean     (JNICALL *ResetStatistic)         (JNIEnv *env,
                                                      jvalue obj,
                                                      jmmStatisticType type);
    
      void         (JNICALL *SetPoolSensor)          (JNIEnv *env,
                                                      jobject pool,
                                                      jmmThresholdType type,
                                                      jobject sensor);
    
      jlong        (JNICALL *SetPoolThreshold)       (JNIEnv *env,
                                                      jobject pool,
                                                      jmmThresholdType type,
                                                      jlong threshold);
      jobject      (JNICALL *GetPoolCollectionUsage) (JNIEnv* env, jobject pool);
    
      jint         (JNICALL *GetGCExtAttributeInfo)  (JNIEnv *env,
                                                      jobject mgr,
                                                      jmmExtAttributeInfo *ext_info,
                                                      jint count);
      void         (JNICALL *GetLastGCStat)          (JNIEnv *env,
                                                      jobject mgr,
                                                      jmmGCStat *gc_stat);
    
      jlong        (JNICALL *GetThreadCpuTimeWithKind)
                                                     (JNIEnv *env,
                                                      jlong thread_id,
                                                      jboolean user_sys_cpu_time);
      void         (JNICALL *GetThreadCpuTimesWithKind)
                                                     (JNIEnv *env,
                                                      jlongArray ids,
                                                      jlongArray timeArray,
                                                      jboolean user_sys_cpu_time);
    
      jint         (JNICALL *DumpHeap0)              (JNIEnv *env,
                                                      jstring outputfile,
                                                      jboolean live);
      jobjectArray (JNICALL *FindDeadlocks)          (JNIEnv *env,
                                                      jboolean object_monitors_only);
      void         (JNICALL *SetVMGlobal)            (JNIEnv *env,
                                                      jstring flag_name,
                                                      jvalue  new_value);
      void*        reserved6;
      jobjectArray (JNICALL *DumpThreads)            (JNIEnv *env,
                                                      jlongArray ids,
                                                      jboolean lockedMonitors,
                                                      jboolean lockedSynchronizers);
      void         (JNICALL *SetGCNotificationEnabled) (JNIEnv *env,
                                                        jobject mgr,
                                                        jboolean enabled);
    } JmmInterface;
```







