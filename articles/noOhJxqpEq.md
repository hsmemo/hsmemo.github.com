---
layout: default
title: UnhandledOops クラスおよびその補助クラス (UnhandledOopEntry, UnhandledOops)
---
[Top](../index.html)

#### UnhandledOops クラスおよびその補助クラス (UnhandledOopEntry, UnhandledOops)

これらは, デバッグ用(開発時用)のクラス (#ifdef CHECK_UNHANDLED_OOPS 時にしか定義されない).
oop が適切に Handle 化されているかどうかを実行時にチェックするためのクラス (See: [here](no2935rfO.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/unhandledOops.hpp))
    #ifdef CHECK_UNHANDLED_OOPS
    
    // Detect unhanded oops in VM code
    
    // The design is that when an oop is declared on the stack as a local
    // variable, the oop is actually a C++ struct with constructor and
    // destructor.  The constructor adds the oop address on a list
    // off each thread and the destructor removes the oop.  At a potential
    // safepoint, the stack addresses of the local variable oops are trashed
    // with a recognizeable value.  If the local variable is used again, it
    // will segfault, indicating an unsafe use of that oop.
    // eg:
    //    oop o;    //register &o on list
    //    funct();  // if potential safepoint - causes clear_naked_oops()
    //              // which trashes o above.
    //    o->do_something();  // Crashes because o is unsafe.
    //
    // This code implements the details of the unhandled oop list on the thread.
```


### クラス一覧(class list)

  * [UnhandledOops](#no669NMwO-)
  * [UnhandledOopEntry](#noeqsICLz2)


---
## <a name="no669NMwO-" id="no669NMwO-">UnhandledOops</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef CHECK_UNHANDLED_OOPS 時にしか定義されない)
(また, develop オプションである CheckUnhandledOops がセットされている場合(= true の場合)でないと使用されない).

あるスレッドが現在 Unhandled Oops Check の検査対象としている oop 全体を管理するためのクラス.


```
    ((cite: hotspot/src/share/vm/runtime/unhandledOops.hpp))
    class UnhandledOops {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Thread オブジェクトの _unhandled_oops フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
Thread::Thread() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classUnhandledOops.html) for details

---
## <a name="noeqsICLz2" id="noeqsICLz2">UnhandledOopEntry</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef CHECK_UNHANDLED_OOPS 時にしか定義されない)
(また, develop オプションである CheckUnhandledOops がセットされている場合(= true の場合)でないと使用されない).

UnhandledOops クラス内で使用される補助クラス.

Unhandled Oops Check の検査対象になっている oop を記憶しておくためのクラス.
1つの UnhandledOopEntry オブジェクトが検査対象の oop 1個に対応する (See: [here](no2935rfO.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/unhandledOops.hpp))
    class UnhandledOopEntry {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 UnhandledOops オブジェクトの _oop_list フィールドに(のみ)格納されている.

(正確には, このフィールドは UnhandledOopEntry の GrowableArray を格納するフィールド.
この中に, その UnhandledOops 内で使用される全ての UnhandledOopEntry オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
GrowableArray 用のメモリ領域は UnhandledOops::UnhandledOops() 内で(のみ)確保されている. 

そのメモリ領域中に個別の UnhandledOopEntry オブジェクトを書き込む作業は, 以下の箇所で(のみ)行われている.

* UnhandledOops::register_unhandled_oop()
* UnhandledOops::allow_unhandled_oop()

### 内部構造(Internal structure)
定義されているフィールドは以下の通り
(破壊対象の oop を示す).


```
    ((cite: hotspot/src/share/vm/runtime/unhandledOops.hpp))
     private:
      oop* _oop_ptr;
      bool _ok_for_gc;
      address _pc;
```




### 詳細(Details)
See: [here](../doxygen/classUnhandledOopEntry.html) for details

---
