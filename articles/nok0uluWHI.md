---
layout: default
title: SignatureIterator クラス関連のクラス (SignatureIterator, SignatureTypeNames, SignatureInfo, ArgumentSizeComputer, ArgumentCount, ResultTypeFinder, Fingerprinter, NativeSignatureIterator, SignatureStream, SignatureVerifier)
---
[Top](../index.html)

#### SignatureIterator クラス関連のクラス (SignatureIterator, SignatureTypeNames, SignatureInfo, ArgumentSizeComputer, ArgumentCount, ResultTypeFinder, Fingerprinter, NativeSignatureIterator, SignatureStream, SignatureVerifier)

これらは, Java の型を表す文字列(Signature String)を扱うためのユーティリティ・クラス.
より具体的に言うと, Signature String に対して iterate 処理するためのイテレータクラス.

(なお Signature String とは "[Lfoo;D)I" みたいな文字列のこと)


### クラス一覧(class list)

  * [SignatureIterator](#noCic41UXZ)
  * [SignatureTypeNames](#no7Z_lXuve)
  * [SignatureInfo](#nog496B73h)
  * [ArgumentSizeComputer](#noXDSfWgR0)
  * [ArgumentCount](#noEbNk8d5F)
  * [ResultTypeFinder](#no69cZIpNu)
  * [Fingerprinter](#notsabKGqs)
  * [NativeSignatureIterator](#noW-mMnntx)
  * [SignatureStream](#noNiE24ZW2)
  * [SignatureVerifier](#noJasqPkB9)


---
## <a name="noCic41UXZ" id="noCic41UXZ">SignatureIterator</a>

### 概要(Summary)
Java の型を表す文字列(Signature String)に対して iterate 処理するための一時オブジェクト(ResourceObjクラス) (の基底クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    class SignatureIterator: public ResourceObj {
```

### 使われ方(Usage)
使用する際には, (iterate 処理で呼び出される) do_int() メソッドや 
do_array() メソッド等をオーバーライドしたサブクラスを作ればいい.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    // SignatureIterators iterate over a Java signature (or parts of it).
    // (Syntax according to: "The Java Virtual Machine Specification" by
    // Tim Lindholm & Frank Yellin; section 4.3 Descriptors; p. 89ff.)
    //
    // Example: Iterating over ([Lfoo;D)I using
    //                         0123456789
    //
    // iterate_parameters() calls: do_array(2, 7); do_double();
    // iterate_returntype() calls:                              do_int();
    // iterate()            calls: do_array(2, 7); do_double(); do_int();
    //
    // is_return_type()        is: false         ; false      ; true
    //
    // NOTE: The new optimizer has an alternate, for-loop based signature
    // iterator implemented in opto/type.cpp, TypeTuple::make().
```




### 詳細(Details)
See: [here](../doxygen/classSignatureIterator.html) for details

---
## <a name="no7Z_lXuve" id="no7Z_lXuve">SignatureTypeNames</a>

### 概要(Summary)
SignatureIterator クラスのサブクラスの1つ.

このクラスは, Signature String 中の各型の型名(を表す文字列)に対して何らかの処理を行う場合用.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    // Specialized SignatureIterators: Used to compute signature specific values.
    
    class SignatureTypeNames : public SignatureIterator {
```

### 使われ方(Usage)
使用する際には, type_name() メソッドをオーバーライドしたサブクラスを作ればいい.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
      virtual void type_name(const char* name)   = 0;
```

### 内部構造(Internal structure)
各 do_*() メソッドでは, その型名を表す文字列を引数として type_name() が呼び出される.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
      void do_bool()                       { type_name("jboolean"); }
      void do_char()                       { type_name("jchar"   ); }
      void do_float()                      { type_name("jfloat"  ); }
      void do_double()                     { type_name("jdouble" ); }
      void do_byte()                       { type_name("jbyte"   ); }
      void do_short()                      { type_name("jshort"  ); }
      void do_int()                        { type_name("jint"    ); }
      void do_long()                       { type_name("jlong"   ); }
      void do_void()                       { type_name("void"    ); }
      void do_object(int begin, int end)   { type_name("jobject" ); }
      void do_array (int begin, int end)   { type_name("jobject" ); }
```

### 備考(Notes)
今のところ SignatureTypeNames のサブクラスはこいつだけ
(しかもこいつは #ifndef PRODUCT でないと定義されないデバッグ用(開発時用)のクラス...).


```
    ((cite: hotspot/src/share/vm/oops/methodOop.cpp))
    #ifndef PRODUCT
    class SignatureTypePrinter : public SignatureTypeNames {
```





### 詳細(Details)
See: [here](../doxygen/classSignatureTypeNames.html) for details

---
## <a name="nog496B73h" id="nog496B73h">SignatureInfo</a>

### 概要(Summary)
SignatureIterator クラスのサブクラスの1つ.

このクラスは, Signature String 中の各型(を表す定数)およびその型のサイズ情報に対して何らかの処理を行う場合用.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    class SignatureInfo: public SignatureIterator {
```

### 使われ方(Usage)
使用する際には, set() メソッドをオーバーライドしたサブクラスを作ればいい.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
      virtual void set(int size, BasicType type) = 0;
```

### 内部構造(Internal structure)
各 do_*() メソッドでは, その型を表す定数と型の大きさを引数として set() が呼び出される.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
      void do_bool  ()                     { set(T_BOOLEAN_size, T_BOOLEAN); }
      void do_char  ()                     { set(T_CHAR_size   , T_CHAR   ); }
      void do_float ()                     { set(T_FLOAT_size  , T_FLOAT  ); }
      void do_double()                     { set(T_DOUBLE_size , T_DOUBLE ); }
      void do_byte  ()                     { set(T_BYTE_size   , T_BYTE   ); }
      void do_short ()                     { set(T_SHORT_size  , T_SHORT  ); }
      void do_int   ()                     { set(T_INT_size    , T_INT    ); }
      void do_long  ()                     { set(T_LONG_size   , T_LONG   ); }
      void do_void  ()                     { set(T_VOID_size   , T_VOID   ); }
      void do_object(int begin, int end)   { set(T_OBJECT_size , T_OBJECT ); }
      void do_array (int begin, int end)   { set(T_ARRAY_size  , T_ARRAY  ); }
```




### 詳細(Details)
See: [here](../doxygen/classSignatureInfo.html) for details

---
## <a name="noXDSfWgR0" id="noXDSfWgR0">ArgumentSizeComputer</a>

### 概要(Summary)
SignatureInfo クラスの具象サブクラスの1つ.

メソッド型を表す Signature String を渡すと, 引数の合計サイズを計算してくれる.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    // Specialized SignatureIterator: Used to compute the argument size.
    
    class ArgumentSizeComputer: public SignatureInfo {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. コード中で ArgumentSizeComputer 型の局所変数を宣言する (コンストラクタ引数でメソッド型を表す Signature String を渡す).
2. ArgumentSizeComputer.size() で引数の合計サイズが取得できる.

(なお局所変数の宣言が煩わしければ, コンストラクタからメソッドチェインして size(), という荒技も可能)

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classArgumentSizeComputer.html) for details

---
## <a name="noEbNk8d5F" id="noEbNk8d5F">ArgumentCount</a>

### 概要(Summary)
SignatureInfo クラスの具象サブクラスの1つ.

メソッド型を表す Signature String を渡すと, 引数の個数を計算してくれる.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    class ArgumentCount: public SignatureInfo {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. コード中で ArgumentCount 型の局所変数を宣言する (コンストラクタ引数でメソッド型を表す Signature String を渡す).
2. ArgumentCount.size() で引数の個数が取得できる.

(なお局所変数の宣言が煩わしければ, コンストラクタからメソッドチェインして size(), という手もある.
 というか, ArgumentSizeComputer とは逆で, ArgumentCount の場合はこちらの用法の方が圧倒的に多い. 似たようなクラスなのに何故違う??)

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classArgumentCount.html) for details

---
## <a name="no69cZIpNu" id="no69cZIpNu">ResultTypeFinder</a>

### 概要(Summary)
SignatureInfo クラスの具象サブクラスの1つ.

メソッド型を表す Signature String を渡すと, 返値の型を取得してくれる.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    // Specialized SignatureIterator: Used to compute the result type.
    
    class ResultTypeFinder: public SignatureInfo {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. コード中で ResultTypeFinder 型の局所変数を宣言する (コンストラクタ引数でメソッド型を表す Signature String を渡す).
2. ResultTypeFinder.type() で返値の型(を表す BasicType 値)が取得できる.

(なお局所変数の宣言が煩わしければ, コンストラクタからメソッドチェインして type(), 
 も出来るはずだがこのクラスでは行われていない)

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* ClassFileParser::parse_method()
* Bytecode_member_ref::result_type()
* methodOopDesc::result_type()
* methodOopDesc::make_invoke_method()
* JvmtiEnvBase::check_top_frame()




### 詳細(Details)
See: [here](../doxygen/classResultTypeFinder.html) for details

---
## <a name="notsabKGqs" id="notsabKGqs">Fingerprinter</a>

### 概要(Summary)
SignatureIterator クラスの具象サブクラスの1つ.

メソッド型を表す Signature String を渡すと, それに対するハッシュ値(finger print)を計算してくれる.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    // Fingerprinter computes a unique ID for a given method. The ID
    // is a bitvector characterizing the methods signature (incl. the receiver).
    class Fingerprinter: public SignatureIterator {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. コード中で Fingerprinter 型の局所変数を宣言する (コンストラクタ引数で methodOop を渡す).
2. Fingerprinter.fingerprint() で, そのメソッドの型に対応するハッシュ値(finger print)が取得できる.

(なお局所変数の宣言が煩わしければ, コンストラクタからメソッドチェインして fingerprint(), という手もある.
 というか, 正確に言うとこのクラスの場合はこの用法しかない (局所変数を宣言している箇所はない))

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* SignatureHandlerLibrary::add()


```
    ((cite: hotspot/src/share/vm/interpreter/interpreterRuntime.cpp))
    void SignatureHandlerLibrary::add(methodHandle method) {
    ...
        if (UseFastSignatureHandlers && method->size_of_parameters() <= Fingerprinter::max_size_of_parameters) {
    ...
          // lookup method signature's fingerprint
          uint64_t fingerprint = Fingerprinter(method).fingerprint();
```

* FingerprintMethodsClosure::do_object()
  
  (...そういえば ResourceObj だったな. new してる箇所が他にないから忘れてたが...)

* jni_invoke_static()


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    static void jni_invoke_static(JNIEnv *env, JavaValue* result, jobject receiver, JNICallType call_type, jmethodID method_id, JNI_ArgumentPusher *args, TRAPS) {
    ...
      // Fill out JavaCallArguments object
      args->iterate( Fingerprinter(method).fingerprint() );
```

* jni_invoke_nonstatic()


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    static void jni_invoke_nonstatic(JNIEnv *env, JavaValue* result, jobject receiver, JNICallType call_type, jmethodID method_id, JNI_ArgumentPusher *args, TRAPS) {
    ...
      // Fill out JavaCallArguments object
      args->iterate( Fingerprinter(method).fingerprint() );
```

### 内部構造(Internal structure)
ハッシュ値の作成方法は「型に応じた定数(大きさはそれぞれ parameter_feature_size ビット)を引数の順に並べる」という単純なもの.

なお parameter_feature_size の値は現在は 4.
ハッシュ値の大きさは 64bit (uint64_t) なので, 内部に含まれる型が 16 個以下であれば一意な finger print になることが保証される.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
      void do_bool()    { _fingerprint |= (((uint64_t)bool_parm) << _shift_count); _shift_count += parameter_feature_size; }
      void do_char()    { _fingerprint |= (((uint64_t)char_parm) << _shift_count); _shift_count += parameter_feature_size; }
      void do_byte()    { _fingerprint |= (((uint64_t)byte_parm) << _shift_count); _shift_count += parameter_feature_size; }
      void do_short()   { _fingerprint |= (((uint64_t)short_parm) << _shift_count); _shift_count += parameter_feature_size; }
      void do_int()     { _fingerprint |= (((uint64_t)int_parm) << _shift_count); _shift_count += parameter_feature_size; }
      void do_long()    { _fingerprint |= (((uint64_t)long_parm) << _shift_count); _shift_count += parameter_feature_size; }
      void do_float()   { _fingerprint |= (((uint64_t)float_parm) << _shift_count); _shift_count += parameter_feature_size; }
      void do_double()  { _fingerprint |= (((uint64_t)double_parm) << _shift_count); _shift_count += parameter_feature_size; }
    
      void do_object(int begin, int end)  { _fingerprint |= (((uint64_t)obj_parm) << _shift_count); _shift_count += parameter_feature_size; }
      void do_array (int begin, int end)  { _fingerprint |= (((uint64_t)obj_parm) << _shift_count); _shift_count += parameter_feature_size; }
    
      void do_void()    { ShouldNotReachHere(); }
```




### 詳細(Details)
See: [here](../doxygen/classFingerprinter.html) for details

---
## <a name="noW-mMnntx" id="noW-mMnntx">NativeSignatureIterator</a>

### 概要(Summary)
SignatureIterator クラスのサブクラスの1つ.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

それぞれの型に応じて pass_int(), pass_double(), pass_object() 等のメソッドが呼ばれる他, 
それらの型のサイズを _offset フィールドと _jni_offset フィールドに累積していく.

なお型の取り扱いは JVM 上での扱い方に合わせており, 
bool や char が pass_int() に引き渡されたり, long や double の場合は _offset が２つ増えたりする.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    // Specialized SignatureIterator: Used for native call purposes
    
    class NativeSignatureIterator: public SignatureIterator {
```

### 使われ方(Usage)
使用する際には, pass_*() メソッドをオーバーライドしたサブクラスを作ればいい.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
      virtual void pass_int()              = 0;
      virtual void pass_long()             = 0;
      virtual void pass_object()           = 0;
      virtual void pass_float()            = 0;
    #ifdef _LP64
      virtual void pass_double()           = 0;
    #else
      virtual void pass_double()           { pass_long(); }  // may be same as long
    #endif
```

### 内部構造(Internal structure)
各 do_*() メソッドでは, 対応する pass_*() メソッドを呼んだ後, 
_offset フィールドと _jni_offset フィールドを適切に増加させる.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
      void do_bool  ()                     { pass_int();    _jni_offset++; _offset++;       }
      void do_char  ()                     { pass_int();    _jni_offset++; _offset++;       }
      void do_float ()                     { pass_float();  _jni_offset++; _offset++;       }
    #ifdef _LP64
      void do_double()                     { pass_double(); _jni_offset++; _offset += 2;    }
    #else
      void do_double()                     { pass_double(); _jni_offset += 2; _offset += 2; }
    #endif
      void do_byte  ()                     { pass_int();    _jni_offset++; _offset++;       }
      void do_short ()                     { pass_int();    _jni_offset++; _offset++;       }
      void do_int   ()                     { pass_int();    _jni_offset++; _offset++;       }
    #ifdef _LP64
      void do_long  ()                     { pass_long();   _jni_offset++; _offset += 2;    }
    #else
      void do_long  ()                     { pass_long();   _jni_offset += 2; _offset += 2; }
    #endif
      void do_void  ()                     { ShouldNotReachHere();                               }
      void do_object(int begin, int end)   { pass_object(); _jni_offset++; _offset++;        }
      void do_array (int begin, int end)   { pass_object(); _jni_offset++; _offset++;        }
```

なお, _offset フィールドと _jni_offset フィールドは, それぞれ以下のような値を格納するフィールド.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    // We need separate JNI and Java offset values because in 64 bit mode,
    // the argument offsets are not in sync with the Java stack.
    // For example a long takes up 1 "C" stack entry but 2 Java stack entries.
      int          _offset;                // The java stack offset
      int          _prepended;             // number of prepended JNI parameters (1 JNIEnv, plus 1 mirror if static)
      int          _jni_offset;            // the current parameter offset, starting with 0
```




### 詳細(Details)
See: [here](../doxygen/classNativeSignatureIterator.html) for details

---
## <a name="noNiE24ZW2" id="noNiE24ZW2">SignatureStream</a>

### 概要(Summary)
Java の型を表す文字列(Signature String)中の各型をたどるためのイテレータクラス(StackObjクラス).

なお, SignatureIterator は内部イテレータだが, こちらは外部イテレータ.
SignatureIterator のようにサブクラスを作ってメソッドをオーバーライドするという使い方ではなく, 
next() メソッドで次の要素を追っていく.


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    // Handy stream for iterating over signature
    
    class SignatureStream : public StackObj {
```

### 使われ方(Usage)
#### 使用例(usage examples)
実際に使用する際にはこんな感じになる.


```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
            for (SignatureStream ss(m->signature()); !ss.at_return_type(); ss.next()) {
              BasicType t = ss.type();
    ...
            }
```

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classSignatureStream.html) for details

---
## <a name="noJasqPkB9" id="noJasqPkB9">SignatureVerifier</a>

### 概要(Summary)
ClassVerifier クラス内で使用される補助クラス.

型を表す文字列 (Signature String) が正しいかどうかを判定する関数を納めた名前空間
(このクラスは AllStatic ではないが (というか何故か StackObj になっているが) static な定義しか持たない).


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
    class SignatureVerifier : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ClassVerifier::verify_method()
* ClassVerifier::verify_field_instructions()
* ClassVerifier::verify_invoke_instructions()

### 内部構造(Internal structure)
定義されている public メソッドは, 以下の通り.

(なお, is_valid_signature() は現状どこからも使用されていない)


```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
      public:
        // Returns true if the symbol is valid method or type signature
        static bool is_valid_signature(Symbol* sig);
    
        static bool is_valid_method_signature(Symbol* sig);
        static bool is_valid_type_signature(Symbol* sig);
```




### 詳細(Details)
See: [here](../doxygen/classSignatureVerifier.html) for details

---
