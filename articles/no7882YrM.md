---
layout: default
title: Serviceability 機能 ： JVMTI ： (おまけ) JPLIS(java agent) の処理  
---
[Up](no1sX8Q67Q.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI ： (おまけ) JPLIS(java agent) の処理  

--- 
## 概要(Summary)
JPLIS 機能は, libinstrument.so という JVMTI agent, 及び sun.instrument.InstrumentationImpl という Java クラスによって実現されている.
ユーザーが指定した java agent の呼び出しは libinstrument.so の 2つの callback から行われる

  * VMInit 用の callback (eventHandlerVMInit())
    
    ここから java agent の "premain()" メソッドが呼び出される.
   
  * ClassFileLoadHook 用の callback (eventHandlerClassFileLoadHook())
   
    ここから java agent の "transform()" メソッドが呼び出される.
     
処理の流れは以下のようになる.

  1. まず, 初期化時に Arguments::parse_each_vm_init_arg() 関数の中で "-javaagent:" オプションが認識される.
     
     この際に Arguments::add_init_agent() が呼び出され, _agentList に "instrument" が追加される.
     
     (これは普通の JVMTI エージェントとほぼ同様 (See: [here](nompWVL4Hp.html) for details))
   
  2. Threads::create_vm_init_agents() 中で, 
     libinstrument.so が JVMTI agent としてダイナミックロードされる.
     
     (これは普通の JVMTI エージェントとほぼ同様 (See: [here](nompWVL4Hp.html) for details))
   
  3. 各 "-javaagent:" オプションについて, Agent_OnLoad() が 1回ずつ呼ばれる.
  
     この中でコマンドラインオプションの情報が詳細にパースされ, 
     _JPLISAgent::mAgentClassName と _JPLISAgent::mOptionsString に格納される.

     さらに, Agent_OnLoad() (の中で呼ばれる initializeJPLISAgent()) によって
     VMInit 用の callback である eventHandlerVMInit() が登録される.
     
  4. VMInit 時のコールバック処理で eventHandlerVMInit() が呼び出される.

     この中で ClassFileLoadHook 用の callback である
     eventHandlerClassFileLoadHook() が登録される.
     
     また, ユーザーが指定した java agent の premain() メソッドの呼び出しも行われる.
     
  5. クラスファイルがロードされると, ClassFileLoadHook 時のコールバック処理で
     eventHandlerClassFileLoadHook() が呼び出される. (See: [here](no2935WjX.html) for details)

     この中で, ユーザーが指定した java agent の transform() メソッドの呼び出しが行われる.
  

## 備考(Notes)
関連するソースコード, 及びビルド用の Makefile は以下のディレクトリにある.

* jdk/src/share/instrument
* jdk/make/java/instrument/Makefile

## 処理の流れ (概要)(Execution Flows : Summary)
### Agent_OnLoad() での処理
<div class="flow-abst"><pre>
Agent_OnLoad()
-&gt; createNewJPLISAgent()
   -&gt; allocateJPLISAgent()
      -&gt; allocate()
   -&gt; initializeJPLISAgent()
-&gt; parseArgumentTail()
-&gt; readAttributes()
-&gt; getAttribute()
-&gt; appendClassPath()
-&gt; ...
-&gt; recordCommandLineData()
</pre></div>

### VMInit コールバックでの処理
<div class="flow-abst"><pre>
eventHandlerVMInit()
-&gt; processJavaStart()
   -&gt; createInstrumentationImpl()
   -&gt; setLivePhaseEventHandlers()
   -&gt; startJavaAgent()
      -&gt; invokeJavaAgentMainMethod()
         -&gt; jni_CallVoidMethod()
            -&gt; (See: <a href="no3059-0k.html">here</a> for details)
               -&gt; sun.instrument.InstrumentationImpl.loadClassAndCallPremain()
                  -&gt; sun.instrument.InstrumentationImpl.loadClassAndStartAgent()
                     -&gt; java.lang.Class.getDeclaredMethod()   (&lt;= premain() メソッドの取得)
                     -&gt; java.lang.reflect.Method.invoke()
                        -&gt; (ユーザーが指定した java agent の premain() メソッドが呼び出される)
   -&gt; deallocateCommandLineData()
</pre></div>

### ClassFileLoadHook コールバックでの処理
<div class="flow-abst"><pre>
eventHandlerClassFileLoadHook()
-&gt; transformClassFile()
   -&gt; jni_CallObjectMethod()
      -&gt; (See: <a href="no3059-0k.html">here</a> for details)
         -&gt; sun.instrument.InstrumentationImpl.transform()
            -&gt; sun.instrument.TransformerManager.transform()
               -&gt; (ユーザーが指定した java agent の transform() メソッドが呼び出される)
</pre></div>

### java.lang.instrument.Instrumentation クラスの処理
#### java.lang.instrument.Instrumentation.addTransformer() の処理
<div class="flow-abst"><pre>
sun.instrument.InstrumentationImpl.addTransformer(ClassFileTransformer transformer)
-&gt; sun.instrument.InstrumentationImpl.addTransformer(ClassFileTransformer transformer, boolean canRetransform)
   -&gt; sun.instrument.TransformerManager.addTransformer()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### Agent_OnLoad()
(#Under Construction)
See: [here](no17766d1Z.html) for details
### createNewJPLISAgent()
(#Under Construction)

### initializeJPLISAgent()
(#Under Construction)

...






