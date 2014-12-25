---
layout: default
title: VMStructs クラス 
---
[Top](../index.html)

#### VMStructs クラス 



---
## <a name="nocOaBCqys" id="nocOaBCqys">VMStructs</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (Serviceability Agent (SA) 機能用のクラス)
(より正確には, そのための機能を納めた名前空間. 
 このクラスは AllStatic ではないが, static なフィールド／メソッドしか持たない).

内部には Serviceability Agent が必要とするデバッグ用情報を格納している
(例えば, C で定義されている HotSpot 内の構造体のデータレイアウト, 等) (See: [here](no7882l1S.html) for details).

なお, デバッグ情報を納めたテーブルは, 現在はソースコード中で明示的に構築している.
コメントによると, 
他にもやり方はあるだろうけど (例えばコンパイル後に debug 情報から抜き出すとか) このやり方には以下のようなメリットがある, 
とのこと.

  * プラットフォームに依存しない (どのプラットフォームでも, あるいは将来的にも, ちゃんと動作することが保証できる)
  * 情報が VM 内部に組み込まれるので, 外部に情報を置く場合に比べて, バージョンずれの問題が出にくい
  * (debug 情報から抜き出す場合は) product 版を作る場合には (debug 情報ありなしで) ２回ビルドする必要が出て面倒くさい
  * 型の情報などは, typedef しているものはコンパイル後には違う型になってしまうので, 分かりづらくなってしまう
    (例えば現在の方式では jlong 型のフィールドは jlong ときちんと記録しておけるが, 
     コンパイル後に情報を取り出そうとすると long とか long long みたいな情報しか取り出せない)


```cpp
    ((cite: hotspot/src/share/vm/runtime/vmStructs.hpp))
    // This table encapsulates the debugging information required by the
    // serviceability agent in order to run. Specifically, we need to
    // understand the layout of certain C data structures (offsets, in
    // bytes, of their fields.)
    //
    // There are alternatives for the design of this mechanism, including
    // parsing platform-specific debugging symbols from a debug build into
    // a program database. While this current mechanism can be considered
    // to be a workaround for the inability to debug arbitrary C and C++
    // programs at the present time, it does have certain advantages.
    // First, it is platform-independent, which will vastly simplify the
    // initial bringup of the system both now and on future platforms.
    // Second, it is embedded within the VM, as opposed to being in a
    // separate program database; experience has shown that whenever
    // portions of a system are decoupled, version skew is problematic.
    // Third, generating a program database, for example for a product
    // build, would probably require two builds to be done: the desired
    // product build as well as an intermediary build with the PRODUCT
    // flag turned on but also compiled with -g, leading to a doubling of
    // the time required to get a serviceability agent-debuggable product
    // build. Fourth, and very significantly, this table probably
    // preserves more information about field types than stabs do; for
    // example, it preserves the fact that a field is a "jlong" rather
    // than transforming the type according to the typedef in jni_md.h,
    // which allows the Java-side code to identify "Java-sized" fields in
    // C++ data structures. If the symbol parsing mechanism was redone
    // using stabs, it might still be necessary to have a table somewhere
    // containing this information.
    //
    // Do not change the sizes or signedness of the integer values in
    // these data structures; they are fixed over in the serviceability
    // agent's Java code (for bootstrapping).
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/vmStructs.hpp))
    // This class is a friend of most classes, to be able to access
    // private fields
    class VMStructs {
```




### 詳細(Details)
See: [here](../doxygen/classVMStructs.html) for details

---
