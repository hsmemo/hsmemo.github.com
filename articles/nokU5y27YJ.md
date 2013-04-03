---
layout: default
title: CompiledIC, CompiledStaticCall クラス関連のクラス (CompiledICInfo, CompiledIC, StaticCallInfo, CompiledStaticCall)
---
[Top](../index.html)

#### CompiledIC, CompiledStaticCall クラス関連のクラス (CompiledICInfo, CompiledIC, StaticCallInfo, CompiledStaticCall)

これらは, JIT コンパイラが生成したメソッド呼び出し処理の最適化用のクラス (See: [here](no7882MiN.html) for details).

### 概要(Summary)
これらのクラスは, JIT が生成したメソッド呼び出し箇所に対して実行時に書き換えを行う
(Inline Cashing を行ったり, 呼び出し先が JIT 生成コードかインタープリタ実行されるコードかに応じて処理を変更したりする).

* CompiledIC : dynamic dispatch が必要な呼び出し用 (invokevirtual, invokeinterface).
* CompiledStaticCall : dynamic dispatch が不要な呼び出し用 (invokestatic, invokespecial, invokevirtual での opt virtual call).

また, この書き換え処理では以下の補助クラスが使用される.

* CompiledICInfo : CompiledIC から使用される補助クラス. 実際の飛び先情報を格納しておくためのもの.
* StaticCallInfo : CompiledStaticCall から使用される補助クラス. 実際の飛び先情報を格納しておくためのもの.



### クラス一覧(class list)

  * [CompiledIC](#noSpGTDlWh)
  * [CompiledICInfo](#noB8Ibv2cG)
  * [CompiledStaticCall](#noSB6Tcj5A)
  * [StaticCallInfo](#noYkRzcF3b)


---
## <a name="noSpGTDlWh" id="noSpGTDlWh">CompiledIC</a>

### 概要(Summary)
JIT コンパイラが生成した virtual call 箇所を動的に書き換えるためのクラス.

(なお, ResourceObj のサブクラスなので, メモリ中にずっと存在しているわけではなく,
 書き換え処理の際にだけ temporaly に作られるオブジェクト)
   
Inline Caching を実現するためのもので, 名前も "compiled inline cache" の略らしい (See: [here](no7882oxz.html) for details).


```
    ((cite: hotspot/src/share/vm/code/compiledIC.hpp))
    class CompiledIC: public ResourceObj {
```

### 使われ方(Usage)
CompiledIC_at() や CompiledIC_before() などがファクトリメソッド
(なお, これらは CompiledIC クラスのメソッドではなく大域で定義されている関数).

call site の program counter を指定すると, そこから情報を抽出して CompiledIC オブジェクトが作られる.


```
    ((cite: hotspot/src/share/vm/code/compiledIC.hpp))
      // conversion (machine PC to CompiledIC*)
      friend CompiledIC* CompiledIC_before(address return_addr);
      friend CompiledIC* CompiledIC_at(address call_site);
      friend CompiledIC* CompiledIC_at(Relocation* call_site);
```

### 備考(Notes)
以下は, call site の state transition. 
また, 括弧内は飛び先に渡す追加引数(token)がどう変わるかを示している.

* JIT 生成直後は null
* monomorphic になると compiledICHolderOop か klassOop を渡すようになる
* その後, ICmiss が起きると(=ディスパッチ先のクラスが複数ある可能性が出てきたら), megamorphic モードになって methodOop を渡すようになる


```
    ((cite: hotspot/src/share/vm/code/compiledIC.hpp))
    //-----------------------------------------------------------------------------
    // The CompiledIC represents a compiled inline cache.
    //
    // In order to make patching of the inline cache MT-safe, we only allow the following
    // transitions (when not at a safepoint):
    //
    //
    //         [1] --<--  Clean -->---  [1]
    //            /       (null)      \
    //           /                     \      /-<-\
    //          /          [2]          \    /     \
    //      Interpreted  ---------> Monomorphic     | [3]
    //  (compiledICHolderOop)        (klassOop)     |
    //          \                        /   \     /
    //       [4] \                      / [4] \->-/
    //            \->-  Megamorphic -<-/
    //                  (methodOop)
    //
    // The text in paranteses () refere to the value of the inline cache receiver (mov instruction)
    //
    // The numbers in square brackets refere to the kind of transition:
    // [1]: Initial fixup. Receiver it found from debug information
    // [2]: Compilation of a method
    // [3]: Recompilation of a method (note: only entry is changed. The klassOop must stay the same)
    // [4]: Inline cache miss. We go directly to megamorphic call.
    //
    // The class automatically inserts transition stubs (using the InlineCacheBuffer) when an MT-unsafe
    // transition is made to a stub.
```




### 詳細(Details)
See: [here](../doxygen/classCompiledIC.html) for details

---
## <a name="noB8Ibv2cG" id="noB8Ibv2cG">CompiledICInfo</a>

### 概要(Summary)
CompiledIC から使用される補助クラス.

実際の飛び先情報を格納しておくために使われる
(CompiledIC は, ここに格納された情報に基づいて書き換えを行う).


```
    ((cite: hotspot/src/share/vm/code/compiledIC.hpp))
    class CompiledICInfo {
```

### 使われ方(Usage)
以下のように使われる.


```
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    methodHandle SharedRuntime::resolve_sub_helper(JavaThread *thread,
                                               bool is_virtual,
                                               bool is_optimized, TRAPS) {
    ...

(1) CompiledIC::compute_monomorphic_entry() で飛び先が計算され, CompiledICInfo オブジェクトに格納される.

        CompiledIC::compute_monomorphic_entry(callee_method, h_klass,
                         is_optimized, static_bound, virtual_call_info,
                         CHECK_(methodHandle()));
    ...

(1) CompiledICInfo オブジェクトに格納した情報を基に, CompiledIC::set_to_monomorphic() で飛び先の書き換えが行われる.

            CompiledIC* inline_cache = CompiledIC_before(caller_frame.pc());
            if (inline_cache->is_clean()) {
              inline_cache->set_to_monomorphic(virtual_call_info);
            }
```

また, SharedRuntime::handle_ic_miss_helper() 内でも同様にして使われている.


```
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    methodHandle SharedRuntime::handle_ic_miss_helper(JavaThread *thread, TRAPS) {
```




### 詳細(Details)
See: [here](../doxygen/classCompiledICInfo.html) for details

---
## <a name="noSB6Tcj5A" id="noSB6Tcj5A">CompiledStaticCall</a>

### 概要(Summary)
JIT コンパイラが生成した static call 箇所を動的に書き換えるためのクラス.

(なお, CompiledStaticCall 自体は NativeCall のサブクラスなので,
何か状態を持ってメモリ中にずっと存在しているわけではなく,
static call site に対する relocation 処理をまとめたユーティリティ・クラスという感じ)


```
    ((cite: hotspot/src/share/vm/code/compiledIC.hpp))
    class CompiledStaticCall: public NativeCall {
```

### 備考(Notes)
以下は, call site の state transition. 

* JIT 生成直後は clean
* 初回の呼び出し時に compilled code か interpreted code に初期化される
  (なお interpreter の場合は, methodOop 引数をセットするために, static_stub というスタブを介してのジャンプにする)


```
    ((cite: hotspot/src/share/vm/code/compiledIC.hpp))
    //-----------------------------------------------------------------------------
    // The CompiledStaticCall represents a call to a static method in the compiled
    //
    // Transition diagram of a static call site is somewhat simpler than for an inlined cache:
    //
    //
    //           -----<----- Clean ----->-----
    //          /                             \
    //         /                               \
    //    compilled code <------------> interpreted code
    //
    //  Clean:            Calls directly to runtime method for fixup
    //  Compiled code:    Calls directly to compiled code
    //  Interpreted code: Calls to stub that set methodOop reference
    //
```




### 詳細(Details)
See: [here](../doxygen/classCompiledStaticCall.html) for details

---
## <a name="noYkRzcF3b" id="noYkRzcF3b">StaticCallInfo</a>

### 概要(Summary)
CompiledStaticCall から使用される補助クラス.

実際の飛び先情報を格納しておくために使われる
(CompiledStaticCall は, ここに格納された情報に基づいて書き換えを行う).


```
    ((cite: hotspot/src/share/vm/code/compiledIC.hpp))
    class StaticCallInfo {
```

### 使われ方(Usage)
以下のように使われる.


```
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    methodHandle SharedRuntime::resolve_sub_helper(JavaThread *thread,
                                               bool is_virtual,
                                               bool is_optimized, TRAPS) {
    ...

(1) CompiledStaticCall::compute_entry() で飛び先が計算され, StaticCallInfo オブジェクトに格納される.

        CompiledStaticCall::compute_entry(callee_method, static_call_info);
    ...

(1) CompiledStaticCall::set() で, StaticCallInfo オブジェクトに格納した情報を基に, 飛び先の書き換えが行われる.

            CompiledStaticCall* ssc = compiledStaticCall_before(caller_frame.pc());
            if (ssc->is_clean()) ssc->set(static_call_info);
```




### 詳細(Details)
See: [here](../doxygen/classStaticCallInfo.html) for details

---
