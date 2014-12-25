---
layout: default
title: Dependencies クラス関連のクラス (Dependencies, Dependencies::DepStream, DepChange, DepChange::ContextStream, 及びそれらの補助クラス(ClassHierarchyWalker))
---
[Top](../index.html)

#### Dependencies クラス関連のクラス (Dependencies, Dependencies::DepStream, DepChange, DepChange::ContextStream, 及びそれらの補助クラス(ClassHierarchyWalker))

これらは, JIT コンパイラがコード生成時に使用した「仮定」を覚えておくためのクラス

(HotSpot 用語としての "dependency" はコードに対する仮定のこと. 
 例えば, このメソッドはオーバーライドされない, このクラスにサブクラスは1つしかない, 等.
 [Glossary] 参照)

新たなクラスのロードやクラスの動的書き換え(class evolution)によって仮定が崩れた場合には脱最適化処理(deopt)が行われる


```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.hpp))
    //** Dependencies represent assertions (approximate invariants) within
    // the class hierarchy.  An example is an assertion that a given
    // method is not overridden; another example is that a type has only
    // one concrete subtype.  Compiled code which relies on such
    // assertions must be discarded if they are overturned by changes in
    // the class hierarchy.  We can think of these assertions as
    // approximate invariants, because we expect them to be overturned
    // very infrequently.  We are willing to perform expensive recovery
    // operations when they are overturned.  The benefit, of course, is
    // performing optimistic optimizations (!) on the object code.
    //
    // Changes in the class hierarchy due to dynamic linking or
    // class evolution can violate dependencies.  There is enough
    // indexing between classes and nmethods to make dependency
    // checking reasonably efficient.
```


### クラス一覧(class list)

  * [Dependencies](#noYOl7-QED)
  * [Dependencies::DepStream](#notB3JDqlB)
  * [DepChange](#noEJHmol2V)
  * [DepChange::ContextStream](#noc4GCLecQ)
  * [ClassHierarchyWalker](#novOIuxws0)


---
## <a name="noYOl7-QED" id="noYOl7-QED">Dependencies</a>

### 概要(Summary)
JIT コンパイル作業中に使用される一時オブジェクト(ResourceObjクラス).

コンパイル中に使用した dependency 情報を蓄えていくためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.hpp))
    class Dependencies: public ResourceObj {
```

### 使われ方(Usage)
コンパイル作業の開始時に新しい Dependencies インスタンスが生成される.
その Dependencies インスタンスは ciEnv 内に格納され, 
その後のコンパイル作業中に使用した仮定を蓄えていく.


```cpp
    ((cite: hotspot/src/share/vm/opto/compile.cpp))
    void Compile::Init(int aliaslevel) {
    ...
      env()->set_dependencies(new Dependencies(env()));
```

そして, 集められた dependencies はコンパイル完了時に nmethod 内にコピーされる.


```cpp
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    nmethod::nmethod(
      methodOop method,
      int nmethod_size,
      int compile_id,
      int entry_bci,
      CodeOffsets* offsets,
      int orig_pc_offset,
      DebugInformationRecorder* debug_info,
      Dependencies* dependencies,
      CodeBuffer *code_buffer,
      int frame_size,
      OopMapSet* oop_maps,
      ExceptionHandlerTable* handler_table,
      ImplicitExceptionTable* nul_chk_table,
      AbstractCompiler* compiler,
      int comp_level
      )
    ...
    {
    ...
        dependencies->copy_to(this);
```




### 詳細(Details)
See: [here](../doxygen/classDependencies.html) for details

---
## <a name="notB3JDqlB" id="notB3JDqlB">Dependencies::DepStream</a>

### 概要(Summary)
nmethod 中に格納されている dependency set をたどるためのイテレータクラス.

利用する際には, 以下のコメント中の "Usage:" のように使う.

(なお, oop は Handle 化されていないので VM 内で使うように, とのこと)


```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.hpp))
      // Use this to iterate over an nmethod's dependency set.
      // Works on new and old dependency sets.
      // Usage:
      //
      // ;
      // Dependencies::DepType dept;
      // for (Dependencies::DepStream deps(nm); deps.next(); ) {
      //   ...
      // }
      //
      // The caller must be in the VM, since oops are not wrapped in handles.
      class DepStream {
```




### 詳細(Details)
See: [here](../doxygen/classDependencies_1_1DepStream.html) for details

---
## <a name="noEJHmol2V" id="noEJHmol2V">DepChange</a>

### 概要(Summary)
dependency 情報が変化したことを表すためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.hpp))
    // A class hierarchy change coming through the VM (under the Compile_lock).
    // The change is structured as a single new type with any number of supers
    // and implemented interface types.  Other than the new type, any of the
    // super types can be context types for a relevant dependency, which the
    // new type could invalidate.
    class DepChange : public StackObj {
```

### 使われ方(Usage)
Universe::flush_dependents_on() 内で使用されている.

(というか, インスタンスが生成されているのはここしかない.
 ここで作られたインスタンスが flush 処理に関するいろんなメソッドに渡されていって使われている模様)


```cpp
    ((cite: hotspot/src/share/vm/memory/universe.cpp))
    // Flushes compiled methods dependent on dependee.
    void Universe::flush_dependents_on(instanceKlassHandle dependee) {
    ...
      DepChange changes(dependee);
    
      // Compute the dependent nmethods
      if (CodeCache::mark_for_deoptimization(changes) > 0) {
        // At least one nmethod has been marked for deoptimization
        VM_Deoptimize op;
        VMThread::execute(&op);
      }
```

### 内部構造(Internal structure)
内部的には, KlassHandle 型のフィールドを一つだけ保持している.

このフィールドには, 変化を引き起こした新しいクラスが格納される
(現状では, dependency の変化は新しいクラス1つによってのみ生じる, ということになっている).

```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.hpp))
      // each change set is rooted in exactly one new type (at present):
      KlassHandle _new_type;
```

この _new_type フィールドには, コンストラクタに渡された引数がセットされる.


```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.hpp))
      // notes the new type, marks it and all its super-types
      DepChange(KlassHandle new_type)
        : _new_type(new_type)
      {
        initialize();
      }
```




### 詳細(Details)
See: [here](../doxygen/classDepChange.html) for details

---
## <a name="noc4GCLecQ" id="noc4GCLecQ">DepChange::ContextStream</a>

### 概要(Summary)
DepChange オブジェクトが表す変化の影響範囲を辿っていくためのイテレータクラス.

利用する際には, 以下のコメント中の "Usage:" のように使う.


```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.hpp))
      // Usage:
      // for (DepChange::ContextStream str(changes); str.next(); ) {
      //   klassOop k = str.klass();
      //   switch (str.change_type()) {
      //     ...
      //   }
      // }
      class ContextStream : public StackObj {
```

### 内部構造(Internal structure)
iterate 処理では, コンストラクタで指定された DepChange オブジェクトの _new_type フィールドのクラスを起点とし,
そこから辿れる super class や super interface を巡っていく.

この処理は以下のようにして行われる.

1. まず, コンストラクタ内で, 指定された DepChange オブジェクトの _new_type フィールドの値を _klass フィールドにコピーする
   (ついでに, 状態は Start_Klass (_new_type が空なら NO_CHANGE)にしておく).


```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.hpp))
        ContextStream(DepChange& changes)
          : _changes(changes)
        { start(); }
```


```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.hpp))
        // start at the beginning:
        void start() {
          klassOop new_type = _changes.new_type();
          _change_type = (new_type == NULL ? NO_CHANGE: Start_Klass);
          _klass = new_type;
          _ti_base = NULL;
          _ti_index = 0;
          _ti_limit = 0;
        }
```

2. 次に, Dependencies::DepStream::next() の中では,
   まず _klass フィールドの transitive_interfaces を取得し, その後は呼び出し毎に super class を辿っていく.
   そして, super class が無くなったら, 取得していた transitive_interfaces を辿っていく.


```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.cpp))
    bool DepChange::ContextStream::next() {
      switch (_change_type) {
      case Start_Klass:             // initial state; _klass is the new type
        _ti_base = instanceKlass::cast(_klass)->transitive_interfaces();
        _ti_index = 0;
        _change_type = Change_new_type;
        return true;
      case Change_new_type:
        // fall through:
        _change_type = Change_new_sub;
      case Change_new_sub:
        // 6598190: brackets workaround Sun Studio C++ compiler bug 6629277
        {
          _klass = instanceKlass::cast(_klass)->super();
          if (_klass != NULL) {
            return true;
          }
        }
        // else set up _ti_limit and fall through:
        _ti_limit = (_ti_base == NULL) ? 0 : _ti_base->length();
        _change_type = Change_new_impl;
      case Change_new_impl:
        if (_ti_index < _ti_limit) {
          _klass = klassOop( _ti_base->obj_at(_ti_index++) );
          return true;
        }
        // fall through:
        _change_type = NO_CHANGE;  // iterator is exhausted
      case NO_CHANGE:
        break;
      default:
        ShouldNotReachHere();
      }
      return false;
    }
```




### 詳細(Details)
See: [here](../doxygen/classDepChange_1_1ContextStream.html) for details

---
## <a name="novOIuxws0" id="novOIuxws0">ClassHierarchyWalker</a>

### 概要(Summary)
Dependencies クラス内で使用される補助クラス.

指定されたクラスのサブクラスをたどっていき, 
dependency を壊しているサブクラス("witness" と呼ばれている)を見つけるためのクラス.

(なお, 既に dependency として考慮されているサブクラス("participants"と呼んでいる)については処理は省略している, とのこと)


```cpp
    ((cite: hotspot/src/share/vm/code/dependencies.cpp))
    // This hierarchy walker inspects subtypes of a given type,
    // trying to find a "bad" class which breaks a dependency.
    // Such a class is called a "witness" to the broken dependency.
    // While searching around, we ignore "participants", which
    // are already known to the dependency.
    class ClassHierarchyWalker {
```




### 詳細(Details)
See: [here](../doxygen/classClassHierarchyWalker.html) for details

---
