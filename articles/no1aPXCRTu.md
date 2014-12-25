---
layout: default
title: PSMarkSweep クラス (PSMarkSweep, 及びその補助クラス(PSAlwaysTrueClosure))
---
[Top](../index.html)

#### PSMarkSweep クラス (PSMarkSweep, 及びその補助クラス(PSAlwaysTrueClosure))

これらは, ParallelScavengeHeap の MarkSweep 処理
(UseParallelOldGC オプションが指定されていない場合の Major GC 処理)で使用される補助クラス(See: [here](no2114YqK.html) for details).


### クラス一覧(class list)

  * [PSMarkSweep](#nosT5v8RMl)
  * [PSAlwaysTrueClosure](#noAOrXWWy7)


---
## <a name="nosT5v8RMl" id="nosT5v8RMl">PSMarkSweep</a>

### 概要(Summary)
ParallelScavengeHeap 用の MarkSweep クラス
(つまり, Mark Sweep Compact 処理で使用される補助関数や Closure クラス等を納めた名前空間(AllStatic クラス)
 (See: MarkSweep)).

シングルスレッドでの Mark-Sweep-Compact 処理を実装している.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.hpp))
    class PSMarkSweep : public MarkSweep {
```

### 使われ方(Usage)
UseParallelOldGC オプションが指定されていない場合,
Major GC 処理はこのクラスの PSMarkSweep::invoke() メソッドが呼び出されることで実行される (See: [here](no2114YqK.html) for details).

なお UseParallelOldGC オプションが指定されている場合は,
このクラスの代わりに PSParallelCompact クラスが使用される
(See: PSParallelCompact).

### 内部構造(Internal structure)
内部には, Mark-Sweep-Compact 処理のための以下のようなメソッドが定義されている.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.hpp))
      static void invoke(bool clear_all_softrefs);
      static void invoke_no_policy(bool clear_all_softrefs);
```

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.hpp))
      // Mark live objects
      static void mark_sweep_phase1(bool clear_all_softrefs);
      // Calculate new addresses
      static void mark_sweep_phase2();
```

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.hpp))
      // Update pointers
      static void mark_sweep_phase3();
      // Move objects to new positions
      static void mark_sweep_phase4();
```

ただし, 処理の肝心な部分は PSMarkSweepDecorator クラスに実装されており,
PSMarkSweep はそちらに丸投げしているだけだったりする (See: PSMarkSweepDecorator).




### 詳細(Details)
See: [here](../doxygen/classPSMarkSweep.html) for details

---
## <a name="noAOrXWWy7" id="noAOrXWWy7">PSAlwaysTrueClosure</a>

### 概要(Summary)
PSMarkSweep::mark_sweep_phase3() 内で使用される補助クラス.

PSMarkSweep 用の AlwaysTrueClosure クラス
(つまり, JNI の Weak Global Handle を辿る処理で使用される Closure.
 名前の通り, どんな場合でも常に true を返す. (See: AlwaysTrueClosure))

(なおコメントによると, 
このクラスは ParallelScavenge 用のソースファイルではなく全 MarkSweep 共通の部分で定義されるべき, とのこと)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.cpp))
    // This should be moved to the shared markSweep code!
    class PSAlwaysTrueClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
always_true という大域変数に(のみ)格納されている.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.cpp))
    static PSAlwaysTrueClosure always_true;
```

#### 使用箇所(where its instances are used)
PSMarkSweep::mark_sweep_phase3() 内で(のみ)使用されている.

(なお, phase1 の中で ReferenceProcessor::process_discovered_references() が呼び出された段階で,
 死んでいる Weak Global Reference については NULL になっている.
 このため, PSAlwaysTrueClosure のような常に true を返すだけの Closure でも,
 生きている Weak Global Reference だけを全て辿ることができる.)

### 内部構造(Internal structure)
名前の通り, do_object_b() は常に true を返す.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.cpp))
      void do_object(oop p) { ShouldNotReachHere(); }
      bool do_object_b(oop p) { return true; }
```

### 備考(Notes)
このクラスは PSMarkSweep 用だが, PSParallelCompact 用にも全く同じクラスが存在している
(See: PSAlwaysTrueClosure).




### 詳細(Details)
See: [here](../doxygen/classPSAlwaysTrueClosure.html) for details

---
