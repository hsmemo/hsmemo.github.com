---
layout: default
title: CVMI 関数用の補助クラス (JVMTraceWrapper, JVMHistogramElement, RegisterArrayForGC, KlassLink)
---
[Top](../index.html)

#### CVMI 関数用の補助クラス (JVMTraceWrapper, JVMHistogramElement, RegisterArrayForGC, KlassLink)

これらは, CVMI の関数を実装するためのクラス.


### クラス一覧(class list)

  * [JVMTraceWrapper](#noff1_Z2X1)
  * [JVMHistogramElement](#noK8S4azBB)
  * [RegisterArrayForGC](#no7sOa3t59)
  * [KlassLink](#noVPhqnY1T)


---
## <a name="noff1_Z2X1" id="noff1_Z2X1">JVMTraceWrapper</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

CVMI 関数が呼び出された際に, その関数名や引数を tty に出力するための一時オブジェクト(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    #ifdef ASSERT
```


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
      class JVMTraceWrapper : public StackObj {
```

### 使われ方(Usage)
JVMWrapper{|2|3|4} マクロ内で(のみ)使用されている.

(なお, JVMWrapper* マクロは #ifdef ASSERT でない場合には空文字列として定義されている.
 以下の #else 以降が #ifdef ASSERT でない場合)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
      #define JVMWrapper(arg1)                    JVMCountWrapper(arg1); JVMTraceWrapper(arg1)
      #define JVMWrapper2(arg1, arg2)             JVMCountWrapper(arg1); JVMTraceWrapper(arg1, arg2)
      #define JVMWrapper3(arg1, arg2, arg3)       JVMCountWrapper(arg1); JVMTraceWrapper(arg1, arg2, arg3)
      #define JVMWrapper4(arg1, arg2, arg3, arg4) JVMCountWrapper(arg1); JVMTraceWrapper(arg1, arg2, arg3, arg4)
    #else
      #define JVMWrapper(arg1)
      #define JVMWrapper2(arg1, arg2)
      #define JVMWrapper3(arg1, arg2, arg3)
      #define JVMWrapper4(arg1, arg2, arg3, arg4)
    #endif
```

そして JVMWrapper マクロは各 CVMI 関数の先頭に置かれている.
このため各 CVMI 関数が呼び出されると JVMTraceWrapper による出力が行われる.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    JVM_ENTRY(jclass, JVM_DefineClass(JNIEnv *env, const char *name, jobject loader, const jbyte *buf, jsize len, jobject pd))
      JVMWrapper2("JVM_DefineClass %s", name);
```

### 内部構造(Internal structure)
行う処理は, コンストラクタで tty->print() を呼んで出力するだけ.

なお, このクラスは (デバッグ時であることに加えて) TraceJVMCalls オプションが指定されている場合にしか働かない.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
        JVMTraceWrapper(const char* format, ...) {
          if (TraceJVMCalls) {
            va_list ap;
            va_start(ap, format);
            tty->print("JVM ");
            tty->vprint_cr(format, ap);
            va_end(ap);
          }
        }
```

### 備考(Notes)
このクラスは CVMI 関数用だが, JNI 関数にも JNITraceWrapper という類似のクラスが存在する.




### 詳細(Details)
See: [here](../doxygen/classJVMTraceWrapper.html) for details

---
## <a name="noK8S4azBB" id="noK8S4azBB">JVMHistogramElement</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

各 CVMI 関数が呼び出された回数を記録する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    #ifdef ASSERT
```


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
      class JVMHistogramElement : public HistogramElement {
```

### 使われ方(Usage)
(なお, 以下の使われ方は HistogramElement のテンプレ通り (See: HistogramElement, Histogram))

#### インスタンスの格納場所(where its instances are stored)
JVMHistogram という大域変数の Histogram オブジェクト内に(のみ)格納されている.

```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
      Histogram* JVMHistogram;
```

#### 生成箇所(where its instances are created)
JVMCountWrapper() マクロ内で(のみ)生成されている.

```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
      #define JVMCountWrapper(arg) \
          static JVMHistogramElement* e = new JVMHistogramElement(arg); \
          if (e != NULL) e->increment_count();  // Due to bug in VC++, we need a NULL check here eventhough it should never happen!
```

そして, このマクロは JVMWrapper マクロ内で(のみ)使用されている.

(なお, JVMWrapper* マクロは #ifdef ASSERT でない場合には空文字列として定義されている.
 以下の #else 以降が #ifdef ASSERT でない場合)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
      #define JVMWrapper(arg1)                    JVMCountWrapper(arg1); JVMTraceWrapper(arg1)
      #define JVMWrapper2(arg1, arg2)             JVMCountWrapper(arg1); JVMTraceWrapper(arg1, arg2)
      #define JVMWrapper3(arg1, arg2, arg3)       JVMCountWrapper(arg1); JVMTraceWrapper(arg1, arg2, arg3)
      #define JVMWrapper4(arg1, arg2, arg3, arg4) JVMCountWrapper(arg1); JVMTraceWrapper(arg1, arg2, arg3, arg4)
    #else
      #define JVMWrapper(arg1)
      #define JVMWrapper2(arg1, arg2)
      #define JVMWrapper3(arg1, arg2, arg3)
      #define JVMWrapper4(arg1, arg2, arg3, arg4)
    #endif
```

そして JVMWrapper マクロは各 CVMI 関数の先頭に置かれている.
このため各 CVMI 関数が呼び出されると JVMHistogramElement による呼び出し回数の記録処理が行われる.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    JVM_ENTRY(jclass, JVM_DefineClass(JNIEnv *env, const char *name, jobject loader, const jbyte *buf, jsize len, jobject pd))
      JVMWrapper2("JVM_DefineClass %s", name);
```

### 備考(Notes)
このクラスは CVMI 関数用だが, JNI 関数にも JNIHistogramElement という類似のクラスが存在する.




### 詳細(Details)
See: [here](../doxygen/classJVMHistogramElement.html) for details

---
## <a name="no7sOa3t59" id="no7sOa3t59">RegisterArrayForGC</a>

### 概要(Summary)
CVMI 関数用 (より具体的に言うと, 
JVM_GetStackAccessControlContext() 用) の補助クラス
(See: JVM_GetStackAccessControlContext()).

ある oop 配列を作業途中のあるスコープの中でだけ JavaThread オブジェクト内に格納しておきたい, という場合に使われる補助クラス.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    class RegisterArrayForGC {
```

(JVM_GetStackAccessControlContext() の返値である
 java.security.AccessControlContext オブジェクト内には
 java.security.ProtectionDomain オブジェクトの配列を入れる必要がある.
 スレッドのスタックを辿って ProtectionDomain オブジェクトを集めた後,
 その ProtectionDomain を入れる配列を確保する時点で GC が起こると,
 せっかく集めた ProtectionDomain オブジェクトへのポインタが不正になる恐れがある.
 このため, これらの ProtectionDomain オブジェクトを JavaThread 内に入れて strong root 扱いにしている模様)

### 内部構造(Internal structure)
コンストラクタで JavaThread::register_array_for_gc() を呼んで指定の JavaThread 内に指定の配列を登録し, 
デストラクタで (同じく JavaThread::register_array_for_gc() を呼んで) 元に戻している.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
      RegisterArrayForGC(JavaThread *thread, GrowableArray<oop>* array)  {
        _thread = thread;
        _thread->register_array_for_gc(array);
      }
    
      ~RegisterArrayForGC() {
        _thread->register_array_for_gc(NULL);
      }
```




### 詳細(Details)
See: [here](../doxygen/classRegisterArrayForGC.html) for details

---
## <a name="noVPhqnY1T" id="noVPhqnY1T">KlassLink</a>

### 概要(Summary)
CVMI 関数 (より具体的に言うと JVM_GetClassContext() 関数) の処理で使用される一時オブジェクト(ResourceObjクラス).
(なお, JVM_GetClassContext() は java.lang.SecurityManager.getClassContext() を実装するための関数.
 (See: JVM_GetClassContext()))


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    // Utility object for collecting method holders walking down the stack
    class KlassLink: public ResourceObj {
```

### 内部構造(Internal structure)
単なる構造体のようなクラス.
内部には以下の2つの public フィールドのみを持つ (そしてメソッドはない).

```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
      KlassHandle klass;
      KlassLink*  next;
```




### 詳細(Details)
See: [here](../doxygen/classKlassLink.html) for details

---
