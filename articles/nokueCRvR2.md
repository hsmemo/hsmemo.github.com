---
layout: default
title: JvmtiAgentThread クラス 
---
[Top](../index.html)

#### JvmtiAgentThread クラス 



---
## <a name="nopgNDvibB" id="nopgNDvibB">JvmtiAgentThread</a>

### 概要(Summary)
JVMTI の関数 (より具体的に言うと, RunAgentThread() 関数) を実装するためのクラス.

RunAgentThread() 関数によって生成されるスレッドを表す (See: [here](no29357ZC.html) for details).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiAgentThread.hpp))
    //
    // class JvmtiAgentThread
    //
    // JavaThread used to wrap a thread started by an agent
    // using the JVMTI method RunAgentThread.
    //
    class JvmtiAgentThread : public JavaThread {
```

(なお, わざわざサブクラスを作らず JavaThread をそのまま使ってもいいような気がするが,
 JVMTI の仕様により RunAgentThread() で作成したスレッドは
 Java レベルのスレッド一覧取得処理で検出されないことになっているため区別されている模様
 (See: is_jvmti_agent_thread()))

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Threads クラスの _thread_list フィールド (static フィールド) に(のみ)格納されている.

(正確には, このフィールドは JavaThread の線形リストを格納するフィールド.
JavaThread オブジェクトは _next フィールドで次の JavaThread オブジェクトを指せる構造になっている.
生成した JavaThread オブジェクトは全てこの線形リスト内に格納されている)

(なお, この線形リストには Threads::add() で登録される)

#### 生成箇所(where its instances are created)
JvmtiEnv::RunAgentThread() 内で(のみ)生成されている (See: [here](no29357ZC.html) for details).

### 内部構造(Internal structure)
なお, 生成されたスレッドのエントリポイントは JvmtiAgentThread::start_function_wrapper() (See: [here](no29357ZC.html) for details).




### 詳細(Details)
See: [here](../doxygen/classJvmtiAgentThread.html) for details

---
