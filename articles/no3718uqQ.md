---
layout: default
title: Serviceability 機能 ： JVMTI 処理の概要
---
[Up](no1sX8Q67Q.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI 処理の概要

--- 
## 参考(for your information)
<http://openjdk.java.net/groups/hotspot/docs/jvmtiImpl.pdf> : JVM Tool Interface (JVM TI) Implementation in HotSpot, Copyright 2007, Sun Microsystems, Inc

## 概要(Summary)
内部構造は以下のようになっている.

### JVMTI エージェントの管理
  * 内部では, AgentLibrary クラスと AgentLibraryList クラスによって管理されている
    (See: AgentLibrary, AgentLibraryList) (See: [here](nompWVL4Hp.html) for details).
    
    AgentLibrary オブジェクトが, 各 JVMTI エージェントを管理する.
    AgentLibraryList オブジェクトは, それら AgentLibrary オブジェクトを束ねておくためのデータ構造.


### JVMTI 関数のエントリポイント

  * エントリポイントとなる関数は jvmtiEnter.cpp(通常時用) や jvmtiEnterTrace.cpp(デバッグ時用) に定義されている.

    より正確には, JVMTI agent が JavaVM->GetEnv() を呼んだときに返される JVMTI env (関数ポインタのテーブル),
    およびそのテーブル内に納められる関数が定義されている.

    (jvmtiEnterTrace.cpp は #ifdef JVMTI_TRACE かつ TraceJVMTI オプションが指定された場合にのみ使われる.
    こちらで定義されている関数にはトレース出力機能が付いている)

  * ただし, jvmtiEnter.cpp 内の jvmti_*() 関数群 (または jvmtiEnterTrace.cpp 内の jvmtiTrace_*() 関数群) は簡単なチェックなどを行っているだけで,
    実際の処理はそこから呼び出される JvmtiEnv クラスの各メソッド(@ jvmtiEnv.cpp)に任されている.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table32740zXZ -->
|  | jvmtiEnter.cpp | jvmtiEnterTrace.cpp |
|---|---|---|
| テーブル | jvmtiInterface_1_ jvmti_Interface | jvmtiInterface_1_ jvmtiTrace_Interface |
| テーブルに納められる関数 | jvmti_${JVMTI 仕様で定められた関数名} という名前の JNICALL 関数 | jvmtiTrace_${JVMTI 仕様で定められた関数名} という名前の JNICALL 関数 |
| 実際の処理を行う関数 | JvmtiEnv クラスの「JVMTI 仕様で定められた関数名」と同名のメソッド | (同左) |
<!-- END RECEIVE ORGTBL table32740zXZ -->

<!-- 
#+ORGTBL: SEND table32740zXZ orgtbl-to-gfm :no-escape t
|                          | jvmtiEnter.cpp                                                    | jvmtiEnterTrace.cpp                                                  |
|--------------------------+-------------------------------------------------------------------+----------------------------------------------------------------------|
| テーブル                 | jvmtiInterface_1_ jvmti_Interface                                 | jvmtiInterface_1_ jvmtiTrace_Interface                               |
| テーブルに納められる関数 | jvmti_${JVMTI 仕様で定められた関数名} という名前の JNICALL 関数   | jvmtiTrace_${JVMTI 仕様で定められた関数名} という名前の JNICALL 関数 |
| 実際の処理を行う関数     | JvmtiEnv クラスの「JVMTI 仕様で定められた関数名」と同名のメソッド | (同左)                                                               |
-->

### JVMTI のイベントの通知処理
  * イベント通知は JvmtiExport クラスを介して行われる.

    JvmtiExport クラスは, HotSpot のその他の部分から JVMTI 実装部の内部を隠蔽するためのクラス.
    これにより, その他の部分からは JVMTI 実装内のことは分からないようにしている.

    (その他の部分から JVMTI に働きかけるのはイベント通知時くらい(?)だが, イベントを JVMTI エージェントに通知したい時等には JvmtiExport クラスを介して行う)

### JVMTI に関する内部状態管理
  * JVMTI 処理では, それぞれの JVMTI 環境毎の状態を管理する必要がある
    (どういった capability を取得しているのか,
     NotifyFramePop() でどのフレームが指定されたか, etc).

  * JVMTI に関する JVMTI env やスレッド毎の情報は, 
    「スレッド毎に異なる情報かどうか / JVMTI env 毎に異なる情報かどうか」に応じて,
    3つのクラスで管理されている (JvmtiEnv(※), JvmtiThreadState, JvmtiEnvThreadState).
    
    (※) 実際には, 状態管理系の処理は JvmtiEnv のスーパークラスである JvmtiEnvBase の方に実装されているが...

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table32740mUH -->
|  | Per Thread | All Thread |
|---|---|---|
| Per Environment | JvmtiEnvThreadState | JvmtiEnv(JvmtiEnvBase) |
| All Environment | JvmtiThreadState |  |
<!-- END RECEIVE ORGTBL table32740mUH -->

<!-- 
#+ORGTBL: SEND table32740mUH orgtbl-to-gfm :no-escape t
|                 | Per Thread          | All Thread             |
|-----------------+---------------------+------------------------|
| Per Environment | JvmtiEnvThreadState | JvmtiEnv(JvmtiEnvBase) |
| All Environment | JvmtiThreadState    |                        |
-->

  * 内部実装においては, 各 Thread オブジェクトが対応する JvmtiThreadState オブジェクトへの参照を持っており,
    さらに各 JvmtiThreadState オブジェクトが, 関連する全ての JvmtiEnvThreadState オブジェクトへの参照を持っている.
    
    * JavaThread オブジェクトの _jvmti_thread_state フィールドに JvmtiThreadState オブジェクトが格納されている.

    * 1つのスレッドに関する全ての JvmtiEnvThreadState が, JvmtiThreadState オブジェクト内の線形リストで管理されている
      (リストの先頭は JvmtiThreadState::_head_env_thread_state フィールド.
      そこからつながる JvmtiEnvThreadState オブジェクトの _next フィールドによって線形リストを構成).


```cpp
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
     private:
      JvmtiThreadState *_jvmti_thread_state;
```


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiThreadState.hpp))
      // for support of JvmtiEnvThreadState
      JvmtiEnvThreadState*   _head_env_thread_state;
```

```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvThreadState.hpp))
      JvmtiEnvThreadState *_next;
```




## Subcategories
* [Serviceability 機能 ： JVMTI 処理の概要 ： interp_only_mode  ](no3059eFS.html)



