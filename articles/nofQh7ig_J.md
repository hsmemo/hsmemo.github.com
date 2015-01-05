---
layout: default
title: OptoRuntime クラス, 及び NamedCounter クラス関連のクラス (NamedCounter, BiasedLockingNamedCounter, OptoRuntime)
---
[Top](../index.html)

#### OptoRuntime クラス, 及び NamedCounter クラス関連のクラス (NamedCounter, BiasedLockingNamedCounter, OptoRuntime)

これらは, C2 JIT 生成コード用のランタイム機能を提供するクラス.


### クラス一覧(class list)

  * [OptoRuntime](#nomG07Lkbj)
  * [NamedCounter](#noKmNqG6hC)
  * [BiasedLockingNamedCounter](#no-j9AlYaL)


---
## <a name="nomG07Lkbj" id="nomG07Lkbj">OptoRuntime</a>

### 概要(Summary)
HotSpot 内にある Runtime クラスの1つ 
(= アセンブリでは書くことが難しい複雑な機能(例外ハンドリング, 重量ロック処理, etc)を納めた名前空間(AllStatic クラス))
(See: [here](no1904gX2.html) for details).

その中でも OptoRuntime クラスには (名前の通り) C2 JIT 生成コードから呼び出されるルーチンが納められている.

OptoRuntime は C2 が前提なので「ランタイムのコード自体も Ideal をコンパイルして作る」という凝った作りになっている
(このため1つ1つのメソッドは nmethod として作成され CodeCache で管理されている).
それぞれのメソッドは「JIT の calling convention で呼び出されて対応する C++ の関数を呼び出す」という挙動をする.

(わざわざ Ideal で作ることにどれだけ意味があるか分からないが, 
 JIT からのジャンプ範囲が CodeCache 内に絞れるので 
 sparc で 32bit 即値ジャンプが使える, とか色々メリットがあるんだろう #TODO)


```cpp
    ((cite: hotspot/src/share/vm/opto/runtime.hpp))
    //------------------------------OptoRuntime------------------------------------
    // Opto compiler runtime routines
    //
    // These are all generated from Ideal graphs.  They are called with the
    // Java calling convention.  Internally they call C++.  They are made once at
    // startup time and Opto compiles calls to them later.
    // Things are broken up into quads: the signature they will be called with,
    // the address of the generated code, the corresponding C++ code and an
    // nmethod.
    
    // The signature (returned by "xxx_Type()") is used at startup time by the
    // Generator to make the generated code "xxx_Java".  Opto compiles calls
    // to the generated code "xxx_Java".  When the compiled code gets executed,
    // it calls the C++ code "xxx_C".  The generated nmethod is saved in the
    // CodeCache.  Exception handlers use the nmethod to get the callee-save
    // register OopMaps.
```


```cpp
    ((cite: hotspot/src/share/vm/opto/runtime.hpp))
    class OptoRuntime : public AllStatic {
```

### 内部構造(Internal structure)
定義されているメソッドは以下の通り.

なお, 命名規則は以下の通り.

* xxx_Type()  : それぞれのメソッドについて, JIT から呼ぶ場合の signature を返す関数.
* xxx_Java()  : Ideal から生成されたランタイムコード (JIT の calling convention で呼び出される).
* xxx_C()     : 上記 xxx_Java() から呼び出される C++ 関数.




### 詳細(Details)
See: [here](../doxygen/classOptoRuntime.html) for details

---
## <a name="noKmNqG6hC" id="noKmNqG6hC">NamedCounter</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか使用されない).

C2 JIT コンパイラが生成した各ロック確保操作について, 
実際にそこでロックが行われた回数を蓄積するパフォーマンスカウンタ.
1つの NamedCounter オブジェクトが 1つのロック確保操作に対応する.

なお, このクラスは (デバッグ時であることに加えて) PrintLockStatistics オプションが指定されている場合にしか使用されない.

なお, NamedCounter オブジェクトは JIT コンパイル作業中にだけ使用される一時的なオブジェクトではない
(一度生成されると HotSpot の終了時まで永続的にメモリ上に存在する).


```cpp
    ((cite: hotspot/src/share/vm/opto/runtime.hpp))
    //
    // NamedCounters are tagged counters which can be used for profiling
    // code in various ways.  Currently they are used by the lock coarsening code
    //
    
    class NamedCounter : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* OptoRuntime クラスの _named_counters フィールド (static フィールド)
  
  (正確には, このフィールドは NamedCounter (やそのサブクラス) の線形リストを格納するフィールド.
  NamedCounter オブジェクトは _next フィールドで次の NamedCounter オブジェクトを指せる構造になっている.
  生成した NamedCounter オブジェクトは全てこの線形リスト内に格納されている)
  
* 各 AbstractLockNode オブジェクトの _counter フィールド
   
  その AbstractLockNode のロック確保箇所に対応する NamedCounter オブジェクト
  (格納している NamedCounter オブジェクト自体は, OptoRuntime::_named_counters が指しているものと重複).

#### 生成箇所(where its instances are created)
OptoRuntime::new_named_counter() 内で(のみ)生成されている
(より正確に言うと NamedCounter::LockCounter 定数を引数としてこの関数が呼び出された場合にのみ生成される).

そして, この関数は現在は以下のパスで(のみ) NamedCounter::LockCounter 定数を引数として呼び出されている.

<div class="flow-abst"><pre>
GraphKit::shared_lock()
-&gt; AbstractLockNode::create_lock_counter()  (ただし, #ifndef PRODUCT 時かつ PrintLockStatistics 指定時にしか呼び出されない)
   -&gt; OptoRuntime::new_named_counter()
</pre></div>

#### 情報の記録箇所(where information is recorded)
現在は, GraphKit::increment_counter() が生成するコード内で(のみ)記録されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
GraphKit::shared_lock()
-&gt; GraphKit::increment_counter()
</pre></div>

#### 情報の出力箇所(where the recorded information is output)
現在は, OptoRuntime::print_named_counters() 内で(のみ)出力されている.

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

  * const char *  _name;
    
    この NamedCounter オブジェクトを示す名前. 
    現在は "${class_name}.${method_name}@${bci}" という名前が付けられている (See: OptoRuntime::new_named_counter()).

  * int           _count;
    
    実行回数が記録されるフィールド. JIT 生成コードにはここを指すポインタが埋め込まれる.

  * CounterTag    _tag;
    
    この NamedCounter オブジェクトの種別.
    なお CounterTag は以下の用に定義された enum 値.


```cpp
    ((cite: hotspot/src/share/vm/opto/runtime.hpp))
        enum CounterTag {
        NoTag,
        LockCounter,
        EliminatedLockCounter,
        BiasedLockingCounter
      };
```

  * NamedCounter* _next;
    
    線形リストを構成するためのポインタ. 次の NamedCounter を指す.


```cpp
    ((cite: hotspot/src/share/vm/opto/runtime.hpp))
      const char *  _name;
      int           _count;
      CounterTag    _tag;
      NamedCounter* _next;
```




### 詳細(Details)
See: [here](../doxygen/classNamedCounter.html) for details

---
## <a name="no-j9AlYaL" id="no-j9AlYaL">BiasedLockingNamedCounter</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する diagnostic オプションが指定されている場合にのみ使用される) (See: PrintPreciseBiasedLockingStatistics).

C2 JIT コンパイラが生成した各 fast-lock 操作について, 
実際にそこでロック処理が行われた回数を蓄積するパフォーマンスカウンタ.
1つの NamedCounter オブジェクトが 1つのロック確保操作に対応する.

なお, BiasedLockingNamedCounter オブジェクトは JIT コンパイル作業中にだけ使用される一時的なオブジェクトではない
(一度生成されると HotSpot の終了時まで永続的にメモリ上に存在する).


```cpp
    ((cite: hotspot/src/share/vm/opto/runtime.hpp))
    class BiasedLockingNamedCounter : public NamedCounter {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
OptoRuntime クラスの _named_counters フィールド (static フィールド) に(のみ)格納されている.
  
(正確には, このフィールドは NamedCounter (やそのサブクラス) の線形リストを格納するフィールド.
NamedCounter オブジェクトは _next フィールドで次の ... オブジェクトを指せる構造になっている.
生成した NamedCounter オブジェクトは全てこの線形リスト内に格納されている)
  
#### 生成箇所(where its instances are created)
OptoRuntime::new_named_counter() 内で(のみ)生成されている
(より正確に言うと NamedCounter::BiasedLockingCounter 定数を引数としてこの関数が呼び出された場合にのみ生成される)
そして, この関数は現在は以下のパスで(のみ) NamedCounter::BiasedLockingCounter 定数を引数として呼び出されている.

<div class="flow-abst"><pre>
GraphKit::shared_lock()
-&gt; FastLockNode::create_lock_counter()  (ただし, PrintPreciseBiasedLockingStatistics 指定時にしか呼び出されない)
   -&gt; OptoRuntime::new_named_counter()
</pre></div>

#### 情報の記録箇所(where information is recorded)
現在は, 以下の箇所で(のみ)記録されている.

* x86_64 の場合
  
  enc_class Fast_Lock()

* sparc の場合
  
  MacroAssembler::compiler_lock_object() 

#### 情報の出力箇所(where the recorded information is output)
現在は, OptoRuntime::print_named_counters() 内で(のみ)出力されている.

### 内部構造(Internal structure)
実際の蓄積機能は, BiasedLockingCounters によって実現されている (See: BiasedLockingCounters)


```cpp
    ((cite: hotspot/src/share/vm/opto/runtime.hpp))
      BiasedLockingCounters _counters;
```




### 詳細(Details)
See: [here](../doxygen/classBiasedLockingNamedCounter.html) for details

---
