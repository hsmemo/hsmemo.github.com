---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 局所変数 (Local Variable) ： GetLocal*() 及び SetLocal*() の処理  
---
[Up](noc0EfwnE-.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 局所変数 (Local Variable) ： GetLocal*() 及び SetLocal*() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
* GetLocalObject() の処理

  JvmtiEnv::GetLocalObject()
  -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; VM_GetOrSetLocal::get_java_vframe()
              -&gt; VM_GetOrSetLocal::get_vframe()
           -&gt; VM_GetOrSetLocal::check_slot_type()
        -&gt; VM_GetOrSetLocal::doit()

* GetLocalInstance() の処理

  JvmtiEnv::GetLocalInstance()
  -&gt; VM_GetReceiver::VM_GetReceiver()
     -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, JavaThread* calling_thread, jint depth, int index)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; (同上)
        -&gt; VM_GetOrSetLocal::doit()
           -&gt; (同上)

* GetLocalInt() の処理

  JvmtiEnv::GetLocalInt()
  -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; (同上)
        -&gt; VM_GetOrSetLocal::doit()
           -&gt; (同上)

* GetLocalLong() の処理

  JvmtiEnv::GetLocalLong()
  -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; (同上)
        -&gt; VM_GetOrSetLocal::doit()
           -&gt; (同上)

* GetLocalFloat() の処理

  JvmtiEnv::GetLocalFloat()
  -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; (同上)
        -&gt; VM_GetOrSetLocal::doit()
           -&gt; (同上)

* GetLocalDouble() の処理

  JvmtiEnv::GetLocalDouble()
  -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; (同上)
        -&gt; VM_GetOrSetLocal::doit()
           -&gt; (同上)

* SetLocalObject() の処理

  JvmtiEnv::SetLocalObject()
  -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type, jvalue value)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; (同上)
        -&gt; VM_GetOrSetLocal::doit()
           -&gt; (同上)

* SetLocalInt() の処理

  JvmtiEnv::SetLocalInt()
  -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type, jvalue value)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; (同上)
        -&gt; VM_GetOrSetLocal::doit()
           -&gt; (同上)

* SetLocalLong() の処理

  JvmtiEnv::SetLocalLong()
  -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type, jvalue value)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; (同上)
        -&gt; VM_GetOrSetLocal::doit()
           -&gt; (同上)

* SetLocalFloat() の処理

  JvmtiEnv::SetLocalFloat()
  -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type, jvalue value)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; (同上)
        -&gt; VM_GetOrSetLocal::doit()
           -&gt; (同上)

* SetLocalDouble() の処理

  JvmtiEnv::SetLocalDouble()
  -&gt; VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type, jvalue value)
  -&gt; VMThread::execute()
     -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
        -&gt; VM_GetOrSetLocal::doit_prologue()
           -&gt; (同上)
        -&gt; VM_GetOrSetLocal::doit()
           -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::GetLocalObject()
See: [here](no2935gcg.html) for details
### VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type)
See: [here](no2935UMt.html) for details
### VM_GetOrSetLocal::doit_prologue()
See: [here](no2935TgC.html) for details
### VM_GetOrSetLocal::get_java_vframe()
See: [here](no2935t0O.html) for details
### VM_GetOrSetLocal::get_vframe()
See: [here](no29356-U.html) for details
### VM_GetOrSetLocal::getting_receiver()
See: [here](no2935HJb.html) for details
### VM_GetOrSetLocal::check_slot_type()
(#Under Construction)
See: [here](no2935UTh.html) for details
### VM_GetOrSetLocal::doit()
(#Under Construction)
See: [here](no2935gqI.html) for details
### JvmtiEnv::GetLocalInstance()
See: [here](no2935tmm.html) for details
### VM_GetReceiver::VM_GetReceiver()
See: [here](no2935hdn.html) for details
### VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, JavaThread* calling_thread, jint depth, int index)
See: [here](no2935unt.html) for details
### VM_GetReceiver::getting_receiver()
See: [here](no29357xz.html) for details
### JvmtiEnv::GetLocalInt()
See: [here](no29356ws.html) for details
### JvmtiEnv::GetLocalLong()
See: [here](no2935H7y.html) for details
### JvmtiEnv::GetLocalFloat()
See: [here](no29355EC.html) for details
### JvmtiEnv::GetLocalDouble()
See: [here](no2935GPI.html) for details
### JvmtiEnv::SetLocalObject()
See: [here](no2935TZO.html) for details
### VM_GetOrSetLocal::VM_GetOrSetLocal(JavaThread* thread, jint depth, int index, BasicType type, jvalue value)
See: [here](no2935hWz.html) for details
### JvmtiEnv::SetLocalInt()
See: [here](no2935gjU.html) for details
### JvmtiEnv::SetLocalLong()
See: [here](no2935tta.html) for details
### JvmtiEnv::SetLocalFloat()
See: [here](no293563g.html) for details
### JvmtiEnv::SetLocalDouble()
See: [here](no2935HCn.html) for details






