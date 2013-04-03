---
layout: default
title: VM 内部での例外生成/例外ハンドリングに関するクラス (ThreadShadow, Exceptions, ExceptionMark)
---
[Top](../index.html)

#### VM 内部での例外生成/例外ハンドリングに関するクラス (ThreadShadow, Exceptions, ExceptionMark)

これらは, HotSpot 内での例外オブジェクトの生成処理用/例外ハンドリング処理用のユーティリティ・クラス.


### クラス一覧(class list)

  * [ThreadShadow](#noCMIfbhWV)
  * [Exceptions](#nozFIE46Ot)
  * [ExceptionMark](#nohNGCftiS)


---
## <a name="noCMIfbhWV" id="noCMIfbhWV">ThreadShadow</a>

### 概要(Summary)
Thread クラスの _pending_exception フィールドにアクセスするためのユーティリティ・クラス.

「例外処理はしたいがクラス階層の関係等で Thread クラス自体は参照できない」, というケースで使用する
(See: [here](no2114VSZ.html) for details).


```
    ((cite: hotspot/src/share/vm/utilities/exceptions.hpp))
    // The ThreadShadow class is a helper class to access the _pending_exception
    // field of the Thread class w/o having access to the Thread's interface (for
    // include hierachy reasons).
    
    class ThreadShadow: public CHeapObj {
```

### 使われ方(Usage)
pending_exception 情報にアクセスする際には, 
Thread オブジェクトを ThreadShadow にキャストしてから
pending_exception() メソッド等を呼び出せばいい.


```
    ((cite: hotspot/src/share/vm/utilities/exceptions.hpp))
    #define PENDING_EXCEPTION                        (((ThreadShadow*)THREAD)->pending_exception())
    #define HAS_PENDING_EXCEPTION                    (((ThreadShadow*)THREAD)->has_pending_exception())
    #define CLEAR_PENDING_EXCEPTION                  (((ThreadShadow*)THREAD)->clear_pending_exception())
```

### 備考(Notes)
なお, 現状の実装では Thread クラス自体が ThreadShadow のサブクラスとなっている.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    class Thread: public ThreadShadow {
```




### 詳細(Details)
See: [here](../doxygen/classThreadShadow.html) for details

---
## <a name="nozFIE46Ot" id="nozFIE46Ot">Exceptions</a>

### 概要(Summary)
例外オブジェクトの生成や送出などを行うためのクラス
(より正確には, そのための機能を納めた名前空間. このクラスは AllStatic ではないが, static なメソッドしか持たない)
(See: [here](no2114VSZ.html) for details).

(ただしコメントによると, 
 大抵のケースについては生成/送出用のマクロを用意しているので, 
 それらのマクロで対応できないレアなケースでのみ 
 Exception クラスを直接使うように, 
 とのこと)

(なお, 上記のコメントで言及されているマクロは 
hotspot/src/share/vm/utilities/exceptions.hpp 内で定義されている (See: [here](no3059qOR.html) for details))


```
    ((cite: hotspot/src/share/vm/utilities/exceptions.hpp))
    // Exceptions is a helper class that encapsulates all operations
    // that require access to the thread interface and which are
    // relatively rare. The Exceptions operations should only be
    // used directly if the macros below are insufficient.
    
    class Exceptions {
```




### 詳細(Details)
See: [here](../doxygen/classExceptions.html) for details

---
## <a name="nohNGCftiS" id="nohNGCftiS">ExceptionMark</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (product 版でも使用されているようだが, 実行時検査が失敗した場合に異常終了させるだけのクラスなので, 実質的に意味があるのはデバッグ時のみだと思われる).

「あるコード範囲で例外が発生しない」ということをコード上に明示したい場合に使用される.

(なお, 実際に使用される際には後述の EXCEPTION_MARK マクロという形で使用される)


```
    ((cite: hotspot/src/share/vm/utilities/exceptions.hpp))
    // ExceptionMark is a stack-allocated helper class for local exception handling.
    // It is used with the EXCEPTION_MARK macro.
    
    class ExceptionMark {
```

### 使われ方(Usage)
ExceptionMark オブジェクトを使用するための EXCEPTION_MARK というマクロが用意されている.
「例外が発生しない」ことを明示したいコード箇所に, これを配置すればいい.

このマクロでは, 以下の2つをチェックする.

* そのスコープに入った際に, pending exception が存在していない.
* そのスコープから出る際に, pending exception が存在していない.


```
    ((cite: hotspot/src/share/vm/utilities/exceptions.hpp))
    // Use an EXCEPTION_MARK for 'local' exceptions. EXCEPTION_MARK makes sure that no
    // pending exception exists upon entering its scope and tests that no pending exception
    // exists when leaving the scope.
    
    // See also preserveException.hpp for PRESERVE_EXCEPTION_MARK macro,
    // which preserves pre-existing exceptions and does not allow new
    // exceptions.
    
    #define EXCEPTION_MARK                           Thread* THREAD; ExceptionMark __em(THREAD);
```

### 内部構造(Internal structure)
コンストラクタでカレントスレッドの pending_exception が空であることを確認し, 
デストラクタでもカレントスレッドの pending_exception が空であることを確認するだけ.
(もし空でなければ, fatal() や vm_exit_during_initialization() で HotSpot を異常終了させる.)


```
    ((cite: hotspot/src/share/vm/utilities/exceptions.cpp))
    ExceptionMark::ExceptionMark(Thread*& thread) {
      thread     = Thread::current();
      _thread    = thread;
      if (_thread->has_pending_exception()) {
        oop exception = _thread->pending_exception();
        _thread->clear_pending_exception(); // Needed to avoid infinite recursion
        exception->print();
        fatal("ExceptionMark constructor expects no pending exceptions");
      }
    }
    
    
    ExceptionMark::~ExceptionMark() {
      if (_thread->has_pending_exception()) {
        Handle exception(_thread, _thread->pending_exception());
        _thread->clear_pending_exception(); // Needed to avoid infinite recursion
        if (is_init_completed()) {
          exception->print();
          fatal("ExceptionMark destructor expects no pending exceptions");
        } else {
          vm_exit_during_initialization(exception);
        }
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classExceptionMark.html) for details

---
