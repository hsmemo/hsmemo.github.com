---
layout: default
title: PreserveExceptionMark クラス関連のクラス (PreserveExceptionMark, CautiouslyPreserveExceptionMark, WeakPreserveExceptionMark)
---
[Top](../index.html)

#### PreserveExceptionMark クラス関連のクラス (PreserveExceptionMark, CautiouslyPreserveExceptionMark, WeakPreserveExceptionMark)

これらは, 例外処理のためのクラス (See: [here](no2114VSZ.html) for details).

(なお, 例外処理用の主なクラスは exceptions.hpp に書かれている. このファイルはおまけ的な内容)


```
    ((cite: hotspot/src/share/vm/utilities/preserveException.hpp))
    // This file provides more support for exception handling; see also exceptions.hpp
```

(なおコメントでは, この3つのクラスは全部リファクタリングしたい, みたいなことが書かれていたりするが...)


```
    ((cite: hotspot/src/share/vm/utilities/preserveException.cpp))
    // TODO: These three classes should be refactored
```


### クラス一覧(class list)

  * [PreserveExceptionMark](#nomQQSh59H)
  * [CautiouslyPreserveExceptionMark](#noMHVUeQGP)
  * [WeakPreserveExceptionMark](#noW3xM7lVd)


---
## <a name="nomQQSh59H" id="nomQQSh59H">PreserveExceptionMark</a>

### 概要(Summary)
ソースコード中のあるスコープの間だけ, カレントスレッドの pending_exception の値をリセットしておくためのクラス.


```
    ((cite: hotspot/src/share/vm/utilities/preserveException.hpp))
    class PreserveExceptionMark {
```

### 使われ方(Usage)
PRESERVE_EXCEPTION_MARK マクロ内で(のみ)使用されている.


```
    ((cite: hotspot/src/share/vm/utilities/preserveException.hpp))
    // use global exception mark when allowing pending exception to be set and
    // saving and restoring them
    #define PRESERVE_EXCEPTION_MARK                    Thread* THREAD; PreserveExceptionMark __em(THREAD);
```

そして, PRESERVE_EXCEPTION_MARK マクロ自体は以下の箇所で(のみ)使用されている.

* java_lang_Throwable::fill_in_stack_trace() 内
* Universe::run_finalizers_on_exit() 内
* instanceRefKlass::acquire_pending_list_lock() 内
* instanceRefKlass::release_and_notify_pending_list_lock() 内
* JavaThread::print_frame_layout() 内

### 内部構造(Internal structure)
コンストラクタでカレントスレッドの pending_exception, exception_line, exception_file といった情報を待避し, 
デストラクタで復帰させているだけ.


```
    ((cite: hotspot/src/share/vm/utilities/preserveException.cpp))
    PreserveExceptionMark::PreserveExceptionMark(Thread*& thread) {
      thread     = Thread::current();
      _thread    = thread;
      _preserved_exception_oop = Handle(thread, _thread->pending_exception());
      _thread->clear_pending_exception(); // Needed to avoid infinite recursion
      _preserved_exception_line = _thread->exception_line();
      _preserved_exception_file = _thread->exception_file();
    }
    
    
    PreserveExceptionMark::~PreserveExceptionMark() {
      if (_thread->has_pending_exception()) {
        oop exception = _thread->pending_exception();
        _thread->clear_pending_exception(); // Needed to avoid infinite recursion
        exception->print();
        fatal("PreserveExceptionMark destructor expects no pending exceptions");
      }
      if (_preserved_exception_oop() != NULL) {
        _thread->set_pending_exception(_preserved_exception_oop(), _preserved_exception_file, _preserved_exception_line);
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classPreserveExceptionMark.html) for details

---
## <a name="noMHVUeQGP" id="noMHVUeQGP">CautiouslyPreserveExceptionMark</a>

### 概要(Summary)
特殊な PreserveExceptionMark クラス.

PreserveExceptionMark クラスとほとんど同じだが, 
デストラクタが呼ばれた時点でカレントスレッドに pending_exception があっても異常終了させない.

(PreserveExceptionMark の場合は pending_exception があると異常終了させる.
 CautiouslyPreserveExceptionMark の場合は, 残っていた pending_exception を消し, 待避していた pending_exception に戻すだけ.)


```
    ((cite: hotspot/src/share/vm/utilities/preserveException.hpp))
    // This is a clone of PreserveExceptionMark which asserts instead
    // of failing when what it wraps generates a pending exception.
    // It also addresses bug 6431341.
    class CautiouslyPreserveExceptionMark {
```

### 使われ方(Usage)
?? (このクラスは使用箇所が見当たらない...)




### 詳細(Details)
See: [here](../doxygen/classCautiouslyPreserveExceptionMark.html) for details

---
## <a name="noW3xM7lVd" id="noW3xM7lVd">WeakPreserveExceptionMark</a>

### 概要(Summary)
特殊な PreserveExceptionMark クラス.

PreserveExceptionMark クラスとほとんど同じだが, 
デストラクタが呼ばれた時点でカレントスレッドに pending_exception があっても異常終了させない.

(PreserveExceptionMark の場合は pending_exception があると異常終了させる.
 WeakPreserveExceptionMark の場合は, pending_exception があった場合にはそちらを優先する (待避していた pending_exception には戻さない))


```
    ((cite: hotspot/src/share/vm/utilities/preserveException.hpp))
    // Like PreserveExceptionMark but allows new exceptions to be generated in
    // the body of the mark. If a new exception is generated then the original one
    // is discarded.
    class WeakPreserveExceptionMark {
```

### 使われ方(Usage)
JNI_ENTRY マクロ内で(のみ)使用されている (See: [here](no6897-YH.html) for details).




### 詳細(Details)
See: [here](../doxygen/classWeakPreserveExceptionMark.html) for details

---
