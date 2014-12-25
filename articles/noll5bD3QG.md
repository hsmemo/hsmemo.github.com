---
layout: default
title: DebugInformationRecorder クラス (DebugInformationRecorder, 及びその補助クラス(DIR_Chunk))
---
[Top](../index.html)

#### DebugInformationRecorder クラス (DebugInformationRecorder, 及びその補助クラス(DIR_Chunk))



### クラス一覧(class list)

  * [DebugInformationRecorder](#noofsscyX7)
  * [DIR_Chunk](#nom9aCM4Wx)


---
## <a name="noofsscyX7" id="noofsscyX7">DebugInformationRecorder</a>

### 概要(Summary)
JIT コンパイル作業中に使用される一時オブジェクト(ResourceObjクラス).

生成するコードに関するデバッグ情報を蓄えていくためのクラス
(この情報は GC やスタック辿りや deopt 処理で使用される).


```cpp
    ((cite: hotspot/src/share/vm/code/debugInfoRec.hpp))
    //** The DebugInformationRecorder collects debugging information
    //   for a compiled method.
    //   Debugging information is used for:
    //   - garbage collecting compiled frames
    //   - stack tracing across compiled frames
    //   - deoptimizating compiled frames
```


```cpp
    ((cite: hotspot/src/share/vm/code/debugInfoRec.hpp))
    class DebugInformationRecorder: public ResourceObj {
```

以下のような情報を含んでいる.

  * PcDesc の配列
  * OopRecorder (?)
  * アドレスとOopMapのペア の配列 (というかハッシュ)
  * ScopeDesc 情報 (を蓄えておくための DebugInfoWriteStream)
  * ...(#TODO)

(なお, DebugInformationRecorder 自体は作業用の一時オブジェクト(ResourceObjクラス)なので,
収集された情報は JIT 完了後は nmethod 構造体に格納されて使用される.)

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 以下の処理を JIT 中に行う
   1. 適当な所で add_safepoint() または add_non_safepoint() を呼び出して, その時点での pc に対応した PcDesc を作っていく.
      (なお, add_safepoint() の場合には, 同時に引数で渡した OopMap の登録も行われる)

   2. safepoint に対応する scope 情報を登録する. これは全ての scope に対して以下の処理を実行することで行う. (<= なお, create_scope_values() と create_monitor_values() は, 明示的に構造共有させたいとき用の補助関数と書かれているが?(#TODO))

      1. (必要ならば、) create_scope_values() で locals 情報を登録する
      2. (必要ならば、) create_scope_values() で expressions 情報を登録する
      3. (必要ならば、) create_monitor_values() で monitor stack 情報を登録する
      4. describe_scope() で scope 情報を登録する.

   3. end_safepoint() または end_non_safepoint() を呼び出して, 該当の safepoint に情報を登録し終わったことを通知する.

2. JIT 生成が終了したら, oop_size(), data_size(), pcs_size() で大きさを取得して nmethod 構造体を作り,
   copy_to() メソッドでデータを nmethod 構造体に書き込んでおく.


```cpp
    ((cite: hotspot/src/share/vm/code/debugInfoRec.hpp))
    //   The implementation requires the compiler to use the recorder
    //   in the following order:
    //   1) Describe debug information for safepoints at increasing addresses.
    //      a) Add safepoint entry (use add_safepoint or add_non_safepoint)
    //      b) Describe scopes for that safepoint
    //         - create locals if needed (use create_scope_values)
    //         - create expressions if needed (use create_scope_values)
    //         - create monitor stack if needed (use create_monitor_values)
    //         - describe scope (use describe_scope)
    //         "repeat last four steps for all scopes"
    //         "outer most scope first and inner most scope last"
    //         NB: nodes from create_scope_values and create_locations
    //             can be reused for simple sharing.
    //         - mark the end of the scopes (end_safepoint or end_non_safepoint)
    //   2) Use oop_size, data_size, pcs_size to create the nmethod and
    //      finally migrate the debugging information into the nmethod
    //      by calling copy_to.
```

#### 生成箇所(where its instances are created)
JIT 作業の開始時点である Compile::Init() 処理の中で作成されている.

その後の JIT 作業中で値が蓄えられていく模様(?) #TODO


```cpp
    ((cite: hotspot/src/share/vm/opto/compile.cpp))
    void Compile::Init(int aliaslevel) {
    ...
      env()->set_debug_info(new DebugInformationRecorder(env()->oop_recorder()));
```

#### 使用箇所(where its instances are used)
以下のような場所で呼び出されている.
...(#TODO)

### 内部構造(Internal structure)
以下のようなメソッドを備えている.

  * void DebugInformationRecorder::add_safepoint(int pc_offset, OopMap* map) :
    pc_offset の箇所に対応する OopMap を登録し,
    さらに, pc_offset の箇所に対応する PcDesc を新たに作って登録する.

    * void DebugInformationRecorder::add_oopmap(int pc_offset, OopMap* map) :
      pc_offset の箇所に対応する OopMap を登録するための内部関数.

    * void DebugInformationRecorder::add_new_pc_offset(int pc_offset) :
      pc_offset の箇所に対応する PcDesc を新たに作って登録するための内部関数.
      追加時の PcDesc の値は以下の通り. (pc_offset 以外の値は describe_scope() できちんとしたものに設定する?? #TODO)

          PcDesc(pc_offset, DebugInformationRecorder::serialized_null,
                            DebugInformationRecorder::serialized_null)

  * void DebugInformationRecorder::add_non_safepoint(int pc_offset) :
    pc_offset の箇所に対応する PcDesc を新たに作って登録する.

    (add_oopmap() を呼ばない版の DebugInformationRecorder::add_safepoint().
     あるいは, DebugInformationRecorder::add_new_pc_offset() 呼んでるだけの関数, とも言う...)


  * void DebugInformationRecorder::describe_scope(...) :


      // must call add_safepoint before: it sets PcDesc and this routine uses
      // the last PcDesc set


  * int DebugInformationRecorder::serialize_monitor_values(GrowableArray<MonitorValue*>* monitors) :

  * int DebugInformationRecorder::serialize_scope_values(GrowableArray<ScopeValue*>* values) :




### 詳細(Details)
See: [here](../doxygen/classDebugInformationRecorder.html) for details

---
## <a name="nom9aCM4Wx" id="nom9aCM4Wx">DIR_Chunk</a>

### 概要(Summary)
DebugInformationRecorder クラス内で使用される補助クラス.

なお, "DIR" は DebugInformationRecorder の頭文字.

(#Under Construction)


```cpp
    ((cite: hotspot/src/share/vm/code/debugInfoRec.cpp))
    // Private definition.
    // There is one DIR_Chunk for each scope and values array.
    // A chunk can potentially be used more than once.
    // We keep track of these chunks in order to detect
    // repetition and enable sharing.
    class DIR_Chunk {
```




### 詳細(Details)
See: [here](../doxygen/classDIR__Chunk.html) for details

---
