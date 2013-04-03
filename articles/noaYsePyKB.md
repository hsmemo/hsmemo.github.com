---
layout: default
title: JNI 関数用の補助クラス (DTraceReturnProbeMark_*, JNITraceWrapper, JNIHistogramElement, JNI_ArgumentPusher, JNI_ArgumentPusherVaArg, JNI_ArgumentPusherArray)
---
[Top](../index.html)

#### JNI 関数用の補助クラス (DTraceReturnProbeMark_*, JNITraceWrapper, JNIHistogramElement, JNI_ArgumentPusher, JNI_ArgumentPusherVaArg, JNI_ArgumentPusherArray)

これらは, JNI の機能(より具体的に言うと JNI 関数全般)を実現するためのクラス.


### クラス一覧(class list)

  * [DTraceReturnProbeMark_* (DTraceReturnProbeMark_DefineClass, DTraceReturnProbeMark_FindClass, ...)](#nogeAsnlds)
  * [JNITraceWrapper](#noncrVNqFn)
  * [JNIHistogramElement](#noTI_erNhZ)
  * [JNI_ArgumentPusher](#noPyFlYH5H)
  * [JNI_ArgumentPusherVaArg](#noV3A_PbfE)
  * [JNI_ArgumentPusherArray](#noo9T2F9zq)


---
## <a name="nogeAsnlds" id="nogeAsnlds">DTraceReturnProbeMark_* (DTraceReturnProbeMark_DefineClass, DTraceReturnProbeMark_FindClass, ...)</a>

### 概要(Summary)
保守運用機能のためのクラス (DTrace 用のフック点を管理するためのクラス) (See: [here](no28916ldL.html) for details).

DTrace のフック点(より具体的に言うと JNI 関数のリターン時用のフック点)の生成処理を簡単に行うための補助クラス.
ソースコード上のスコープに連動して自動的にフック点を生成する.

(なお, これらのクラスは DTRACE 機能が有効な場合 (= DTRACE_ONLY() が空文字列に展開されない場合) にのみ定義される)

### 内部構造(Internal structure)
デストラクタで DTrace のフック処理を呼び出すだけ.

### 備考(Notes)
DTraceReturnProbeMark_* はソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.

* DT_RETURN_MARK_DECL() : 返値が void 以外の JNI 関数用
* DT_VOID_RETURN_MARK_DECL() : 返値が void の JNI 関数用

また, 使用する際にも以下のマクロを使って間接的に使用される.

* DT_RETURN_MARK() : 返値が void 以外のもの用
* DT_VOID_RETURN_MARK() : 返値が void のもの用

またこれらのラッパーとして, 以下のマクロも用意されている (型に応じて void 用とそれ以外用を使い分ける).

* DT_RETURN_MARK_DECL_FOR()

  (返値が float/double なら DT_RETURN_MARK_DECL(), それ以外なら DT_VOID_RETURN_MARK_DECL() に展開される)

* DT_RETURN_MARK_FOR()

  (返値が float/double なら DT_RETURN_MARK(), それ以外なら DT_VOID_RETURN_MARK() に展開される)


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    // The DT_RETURN_MARK macros create a scoped object to fire the dtrace
    // '-return' probe regardless of the return path is taken out of the function.
    // Methods that have multiple return paths use this to avoid having to
    // instrument each return path.  Methods that use CHECK or THROW must use this
    // since those macros can cause an immedate uninstrumented return.
    //
    // In order to get the return value, a reference to the variable containing
    // the return value must be passed to the contructor of the object, and
    // the return value must be set before return (since the mark object has
    // a reference to it).
    //
    // Example:
    // DT_RETURN_MARK_DECL(SomeFunc, int);
    // JNI_ENTRY(int, SomeFunc, ...)
    //   int return_value = 0;
    //   DT_RETURN_MARK(SomeFunc, int, (const int&)return_value);
    //   foo(CHECK_0)
    //   return_value = 5;
    //   return return_value;
    // JNI_END
```

#### 参考(for your information): DT_RETURN_MARK_DECL()

```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    #define DT_RETURN_MARK_DECL(name, type)                                    \
      HS_DTRACE_PROBE_DECL1(hotspot_jni, name##__return, type);                \
      DTRACE_ONLY(                                                             \
        class DTraceReturnProbeMark_##name {                                   \
         public:                                                               \
          const type& _ret_ref;                                                \
          DTraceReturnProbeMark_##name(const type& v) : _ret_ref(v) {}         \
          ~DTraceReturnProbeMark_##name() {                                    \
            HS_DTRACE_PROBE1(hotspot_jni, name##__return, _ret_ref);           \
          }                                                                    \
        }                                                                      \
      )
```

#### 参考(for your information): DT_VOID_RETURN_MARK_DECL()

```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    // Void functions are simpler since there's no return value
    #define DT_VOID_RETURN_MARK_DECL(name)                                     \
      HS_DTRACE_PROBE_DECL0(hotspot_jni, name##__return);                      \
      DTRACE_ONLY(                                                             \
        class DTraceReturnProbeMark_##name {                                   \
         public:                                                               \
          ~DTraceReturnProbeMark_##name() {                                    \
            HS_DTRACE_PROBE0(hotspot_jni, name##__return);                     \
          }                                                                    \
        }                                                                      \
      )
```

#### 参考(for your information): DT_RETURN_MARK(), DT_VOID_RETURN_MARK()

```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    // Place these macros in the function to mark the return.  Non-void
    // functions need the type and address of the return value.
    #define DT_RETURN_MARK(name, type, ref) \
      DTRACE_ONLY( DTraceReturnProbeMark_##name dtrace_return_mark(ref) )
    #define DT_VOID_RETURN_MARK(name) \
      DTRACE_ONLY( DTraceReturnProbeMark_##name dtrace_return_mark )
```

#### 参考(for your information): DT_RETURN_MARK_DECL_FOR(), DT_RETURN_MARK_FOR()

```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    // Choose DT_RETURN_MARK macros  based on the type: float/double -> void
    // (dtrace doesn't do FP yet)
    #define DT_RETURN_MARK_DECL_FOR(TypeName, name, type) \
      FP_SELECT(TypeName, \
        DT_RETURN_MARK_DECL(name, type), DT_VOID_RETURN_MARK_DECL(name) )
    #define DT_RETURN_MARK_FOR(TypeName, name, type, ref) \
      FP_SELECT(TypeName, \
        DT_RETURN_MARK(name, type, ref), DT_VOID_RETURN_MARK(name) )
```


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    // Use these to select distinct code for floating-point vs. non-floating point
    // situations.  Used from within common macros where we need slightly
    // different behavior for Float/Double
    #define FP_SELECT_Boolean(intcode, fpcode) intcode
    #define FP_SELECT_Byte(intcode, fpcode)    intcode
    #define FP_SELECT_Char(intcode, fpcode)    intcode
    #define FP_SELECT_Short(intcode, fpcode)   intcode
    #define FP_SELECT_Object(intcode, fpcode)  intcode
    #define FP_SELECT_Int(intcode, fpcode)     intcode
    #define FP_SELECT_Long(intcode, fpcode)    intcode
    #define FP_SELECT_Float(intcode, fpcode)   fpcode
    #define FP_SELECT_Double(intcode, fpcode)  fpcode
    #define FP_SELECT(TypeName, intcode, fpcode) \
      FP_SELECT_##TypeName(intcode, fpcode)
```




### 詳細(Details)
See: [here](../doxygen/classDTraceReturnProbeMark__* (DTraceReturnProbeMark__DefineClass, DTraceReturnProbeMark__FindClass, ...).html) for details

---
## <a name="noncrVNqFn" id="noncrVNqFn">JNITraceWrapper</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

JNI 関数が呼び出された際に, その関数名や引数を tty に出力するための一時オブジェクト(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    #ifdef ASSERT
```


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
      class JNITraceWrapper : public StackObj {
```

### 使われ方(Usage)
JNIWrapper マクロ内で(のみ)使用されている.

(なお, JNIWrapper マクロは #ifdef ASSERT でない場合には空文字列として定義されている.
 以下の #else 以降が #ifdef ASSERT でない場合)


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
      #define JNIWrapper(arg) JNICountWrapper(arg); JNITraceWrapper(arg)
    #else
      #define JNIWrapper(arg)
```

そして, JNIWrapper マクロは各 JNI 関数の先頭に置かれている.
このため各 JNI 関数が呼び出されると JNITraceWrapper による出力が行われる.


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    JNI_ENTRY(jclass, jni_DefineClass(JNIEnv *env, const char *name, jobject loaderRef,
                                      const jbyte *buf, jsize bufLen))
      JNIWrapper("DefineClass");
```

### 内部構造(Internal structure)
行う処理は, コンストラクタで tty->print() を呼んで出力するだけ.

なお, このクラスは (デバッグ時であることに加えて) TraceJNICalls オプションが指定されている場合にしか働かない.


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
        JNITraceWrapper(const char* format, ...) {
          if (TraceJNICalls) {
            va_list ap;
            va_start(ap, format);
            tty->print("JNI ");
            tty->vprint_cr(format, ap);
            va_end(ap);
          }
        }
```

### 備考(Notes)
このクラスは JNI 関数用だが, CVMI 関数にも JVMTraceWrapper という類似のクラスが存在する.




### 詳細(Details)
See: [here](../doxygen/classJNITraceWrapper.html) for details

---
## <a name="noTI_erNhZ" id="noTI_erNhZ">JNIHistogramElement</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

各 JNI 関数が呼び出された回数を記録する.


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    #ifdef ASSERT
```


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
      class JNIHistogramElement : public HistogramElement {
```

### 使われ方(Usage)
(なお, 以下の使われ方は HistogramElement のテンプレ通り (See: HistogramElement, Histogram))

#### インスタンスの格納場所(where its instances are stored)
JNIHistogram という大域変数の Histogram オブジェクト内に(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
      Histogram* JNIHistogram;
```

#### 生成箇所(where its instances are created)
JNICountWrapper マクロ内で(のみ)生成されている.


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
      #define JNICountWrapper(arg)                                     \
         static JNIHistogramElement* e = new JNIHistogramElement(arg); \
          /* There is a MT-race condition in VC++. So we need to make sure that that e has been initialized */ \
         if (e != NULL) e->increment_count()
```

そして, このマクロは JNIWrapper マクロ内で(のみ)使用されている.

(なお, JNIWrapper マクロは #ifdef ASSERT でない場合には空文字列として定義されている.
 以下の #else 以降が #ifdef ASSERT でない場合)


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
      #define JNIWrapper(arg) JNICountWrapper(arg); JNITraceWrapper(arg)
    #else
      #define JNIWrapper(arg)
```

そして, JNIWrapper マクロは各 JNI 関数の先頭に置かれている.
このため各 JNI 関数が呼び出されると JNIHistogramElement による呼び出し回数の記録処理が行われる.


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    JNI_ENTRY(jclass, jni_DefineClass(JNIEnv *env, const char *name, jobject loaderRef,
                                      const jbyte *buf, jsize bufLen))
      JNIWrapper("DefineClass");
```

### 備考(Notes)
このクラスは JNI 関数用だが, CVMI 関数にも JVMHistogramElement という類似のクラスが存在する.




### 詳細(Details)
See: [here](../doxygen/classJNIHistogramElement.html) for details

---
## <a name="noPyFlYH5H" id="noPyFlYH5H">JNI_ArgumentPusher</a>

### 概要(Summary)
JNI の関数 (より具体的に言うと, Call*Method*() 関数および NewObject*() 関数) を実装するためのクラス (の基底クラス).

HotSpot 内部の ABI に従うように引数を詰め直す処理を行う (See: [here](no2935TLm.html) and [here](no3059-0k.html) for details).


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    class JNI_ArgumentPusher : public SignatureIterator {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
      virtual void get_bool   () = 0;
      virtual void get_char   () = 0;
      virtual void get_short  () = 0;
      virtual void get_byte   () = 0;
      virtual void get_int    () = 0;
      virtual void get_long   () = 0;
      virtual void get_float  () = 0;
      virtual void get_double () = 0;
      virtual void get_object () = 0;
```




### 詳細(Details)
See: [here](../doxygen/classJNI__ArgumentPusher.html) for details

---
## <a name="noV3A_PbfE" id="noV3A_PbfE">JNI_ArgumentPusherVaArg</a>

### 概要(Summary)
JNI_ArgumentPusher クラスの具象サブクラスの1つ.

このクラスは, 関数名の最後が 'A' で終わらない JNI 関数用 
(つまり, 引数を va_list に入れて渡す関数 or 関数自体が可変長引数関数となっている関数用).


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    class JNI_ArgumentPusherVaArg : public JNI_ArgumentPusher {
```

### 使われ方(Usage)
以下の JNI 関数の処理で(のみ)使用されている (See: [here](no2935TLm.html) and [here](no3059-0k.html) for details).

* NewObject()
* NewObjectV()
* Call<type>Method()
* Call<type>MethodV()
* CallNonvirtual<type>Method()
* CallNonvirtual<type>MethodV()
* CallStatic<type>Method()
* CallStatic<type>MethodV()




### 詳細(Details)
See: [here](../doxygen/classJNI__ArgumentPusherVaArg.html) for details

---
## <a name="noo9T2F9zq" id="noo9T2F9zq">JNI_ArgumentPusherArray</a>

### 概要(Summary)
JNI_ArgumentPusher クラスの具象サブクラスの1つ.

このクラスは, 関数名の最後が 'A' で終わる JNI 関数用 (つまり, 引数を jvalue の配列に入れて渡す関数用).


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    class JNI_ArgumentPusherArray : public JNI_ArgumentPusher {
```

### 使われ方(Usage)
以下の JNI 関数の処理で(のみ)使用されている (See: [here](no2935TLm.html) and [here](no3059-0k.html) for details).

* NewObjectA()
* Call<type>MethodA()
* CallNonvirtual<type>MethodA()
* CallStatic<type>MethodA()




### 詳細(Details)
See: [here](../doxygen/classJNI__ArgumentPusherArray.html) for details

---
