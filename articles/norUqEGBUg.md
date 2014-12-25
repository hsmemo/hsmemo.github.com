---
layout: default
title: JvmtiCodeBlobEvents クラス (JvmtiCodeBlobEvents, 及びその補助クラス(CodeBlobCollector))
---
[Top](../index.html)

#### JvmtiCodeBlobEvents クラス (JvmtiCodeBlobEvents, 及びその補助クラス(CodeBlobCollector))

これらは, JVMTI の関数を実装するために使われているクラス.
より具体的に言うと, GenerateEvents() 関数を実装するためのクラス (See: [here](no2935lCe.html) for details).


### クラス一覧(class list)

  * [JvmtiCodeBlobEvents](#nofHbzWdXB)
  * [CodeBlobCollector](#no2BA88Mm4)


---
## <a name="nofHbzWdXB" id="nofHbzWdXB">JvmtiCodeBlobEvents</a>

### 概要(Summary)
JVMTI の関数 (より具体的に言うと, GenerateEvents() 関数) のための関数を納めた名前空間(AllStatic クラス).

大きく分けると, 以下の2つの機能を提供している.

  * JVMTI の GenerateEvents() 関数の実装

    (CompiledMethodLoad イベントの再生成処理, DynamicCodeGenerated イベントの再生成処理)

  * GenerateEvents() 関数用のユーティリティ関数 (JvmtiCodeBlobEvents::build_jvmti_addr_location_map())
    	
    (CompiledMethodLoad イベントのハンドラに渡す jvmtiAddrLocationMap 型引数を計算する)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiCodeBlobEvents.hpp))
    // JVMTI code blob event support
    // -- used by GenerateEvents to generate CompiledMethodLoad and
    //    DynamicCodeGenerated events
    // -- also provide utility function build_jvmti_addr_location_map to create
    //    a jvmtiAddrLocationMap list for a nmethod.
    
    class JvmtiCodeBlobEvents : public AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JvmtiEnv::GenerateEvents()

  引数に応じて, JvmtiCodeBlobEvents::generate_compiled_method_load_events() または
  JvmtiCodeBlobEvents::generate_dynamic_code_events() のどちらかが呼び出され,
  CompiledMethodLoad イベント/DynamicCodeGenerated イベントの再生成処理が行われる (See: [here](no2935lCe.html) for details).

* JvmtiCompiledMethodLoadEventMark::JvmtiCompiledMethodLoadEventMark()

  JvmtiCompiledMethodLoadEventMark は,
  CompiledMethodLoad イベントを送出する JvmtiExport::post_compiled_method_load() 関数内で(のみ)使用されているクラス.
  内部で JvmtiCodeBlobEvents::build_jvmti_addr_location_map() を呼び出してハンドラへの引数の計算を行う.
  (See: JvmtiCompiledMethodLoadEventMark)

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.

  * generate_dynamic_code_events() は, JVMTI の GenerateEvents() の実装用.

    nmethod 以外の全ての CodeBlob, および全ての StubCode について, DYNAMIC_CODE_GENERATED_EVENT イベントを生成する.

  * generate_compiled_method_load_events() も, JVMTI の GenerateEvents() の実装用.

    全ての nmethod について, COMPILED_METHOD_LOAD イベントを生成する.

  * build_jvmti_addr_location_map() は, 
    JVMTI の CompiledMethodLoad イベントのハンドラに渡す jvmtiAddrLocationMap 型の引数の値 (およびその長さ)を計算する補助関数.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiCodeBlobEvents.hpp))
      // generate a DYNAMIC_CODE_GENERATED_EVENT event for each non-nmethod
      // code blob in the code cache.
      static jvmtiError generate_dynamic_code_events(JvmtiEnv* env);
    
      // generate a COMPILED_METHOD_LOAD event for each nmethod
      // code blob in the code cache.
      static jvmtiError generate_compiled_method_load_events(JvmtiEnv* env);
    
      // create a C-heap allocated address location map for an nmethod
      static void build_jvmti_addr_location_map(nmethod *nm, jvmtiAddrLocationMap** map,
                                                jint *map_length);
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiCodeBlobEvents.html) for details

---
## <a name="no2BA88Mm4" id="no2BA88Mm4">CodeBlobCollector</a>

### 概要(Summary)
JvmtiCodeBlobEvents クラス内で使用される補助クラス(StackObjクラス).

全ての StubCodeと, CodeCache 内の nmethod 以外の全ての CodeBlob の情報を集める.

なお, CodeCache 内の情報を集めるには CodeCache::blobs_do() を使っている.
しかし, 型を見れば分かるように, 
CodeCache::blobs_do() の iterate 処理に使う関数には自由変数的な引数が渡せない
(引数は CodeBlob 型のもの1つしか渡せない).
そのため, 情報を蓄える先を static 変数にして CodeCache::blobs_do() 内からでも見えるようにしている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiCodeBlobEvents.cpp))
    // Support class to collect a list of the non-nmethod CodeBlobs in
    // the CodeCache.
    //
    // This class actually creates a list of JvmtiCodeBlobDesc - each JvmtiCodeBlobDesc
    // describes a single CodeBlob in the CodeCache. Note that collection is
    // done to a static list - this is because CodeCache::blobs_do is defined
    // as void CodeCache::blobs_do(void f(CodeBlob* nm)) and hence requires
    // a C or static method.
    //
    // Usage :-
    //
    // CodeBlobCollector collector;
    //
    // collector.collect();
    // JvmtiCodeBlobDesc* blob = collector.first();
    // while (blob != NULL) {
    //   :
    //   blob = collector.next();
    // }
    //
    
    class CodeBlobCollector : StackObj {
```

### 使われ方(Usage)
JvmtiCodeBlobEvents::generate_dynamic_code_events() 内で(のみ)使用されている (See: [here](no2935lCe.html) for details).

### 内部構造(Internal structure)
以下の _global_code_blobs が, 調査中に一時的に情報を蓄えていくための static な配列.

最終的な計算結果は _code_blobs という配列に蓄えられる.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiCodeBlobEvents.cpp))
      GrowableArray<JvmtiCodeBlobDesc*>* _code_blobs;   // collected blobs
      int _pos;                                         // iterator position
    
      // used during a collection
      static GrowableArray<JvmtiCodeBlobDesc*>* _global_code_blobs;
      static void do_blob(CodeBlob* cb);
```




### 詳細(Details)
See: [here](../doxygen/classCodeBlobCollector.html) for details

---
