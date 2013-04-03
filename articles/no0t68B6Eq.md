---
layout: default
title: Forte クラス関連のクラス (Forte, vframeStreamForte)
---
[Top](../index.html)

#### Forte クラス関連のクラス (Forte, vframeStreamForte)

これらは, 保守運用機能のためのクラス.
より具体的に言うと, Forte のパフォーマンスアナライザと連携するためのクラス.

(なお, "Forte" は Sun Microsystems 社の開発ツールのブランド名. NetBeans ベースの IDE やコンパイラ, デバッガ, パフォーマンス解析ツール等が含まれる.)


### クラス一覧(class list)

  * [Forte](#no1Hcj5yGz)
  * [vframeStreamForte](#no1c2-bBis)


---
## <a name="no1Hcj5yGz" id="no1Hcj5yGz">Forte</a>

### 概要(Summary)
保守運用機能のためのクラス (Forte との連携機能用のクラス).

Forte::register_stub() というユーティリティ・メソッド(のみ)を納めた名前空間(AllStatic クラス)
(これ以外にメソッドやフィールドはない).


```
    ((cite: hotspot/src/share/vm/prims/forte.hpp))
    // Interface to Forte support.
    
    class Forte : AllStatic {
```

Forte::register_stub() は, 動的に生成したコードをパフォーマンスアナライザに登録する関数
(内部的には collector_func_load() 関数を呼んでいるだけ.
なお collector_func_load() が存在しない環境では呼び出しが NOP になるようにマクロで定義されている.).

#### 参考(for your information): Forte::register_stub()
See: [here](no52482Gk.html) for details
#### 参考(for your information): collector_func_load()

```
    ((cite: hotspot/src/share/vm/prims/forte.cpp))
    #ifndef _WINDOWS
    // Support for the Forte(TM) Peformance Tools collector.
    //
    // The method prototype is derived from libcollector.h. For more
    // information, please see the libcollect man page.
    
    // Method to let libcollector know about a dynamically loaded function.
    // Because it is weakly bound, the calls become NOP's when the library
    // isn't present.
    void    collector_func_load(char* name,
                                void* null_argument_1,
                                void* null_argument_2,
                                void *vaddr,
                                int size,
                                int zero_argument,
                                void* null_argument_3);
    #pragma weak collector_func_load
    #define collector_func_load(x0,x1,x2,x3,x4,x5,x6) \
            ( collector_func_load ? collector_func_load(x0,x1,x2,x3,x4,x5,x6),0 : 0 )
    #endif // !_WINDOWS
```




### 詳細(Details)
See: [here](../doxygen/classForte.html) for details

---
## <a name="no1c2-bBis" id="no1c2-bBis">vframeStreamForte</a>

### 概要(Summary)
保守運用機能のためのクラス (Forte との連携機能用のクラス).

スタックフレーム中のフレームをたどるためのイテレータクラス(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/prims/forte.cpp))
    #ifndef IA64
    
    class vframeStreamForte : public vframeStreamCommon {
```

### 使われ方(Usage)
forte_fill_call_trace_given_top() 内で(のみ)使用されている.
そして, この関数は AsyncGetCallTrace() 内で(のみ)呼び出されている.

なお, AsyncGetCallTrace() はパフォーマンスアナライザによって (SIGPROF のハンドラ等から) 呼び出されることを想定した関数.
スレッドのスタックトレースを取得する
(名前の通り JVMPI の GetCallTrace() 関数 (= JVMTI の GetStackTrace() 関数) の asynchronous 版).




### 詳細(Details)
See: [here](../doxygen/classvframeStreamForte.html) for details

---
