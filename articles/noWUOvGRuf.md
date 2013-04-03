---
layout: default
title: InterpreterRuntime クラス関連のクラス (InterpreterRuntime, SignatureHandlerLibrary, 及びそれらの補助クラス(UnlockFlagSaver))
---
[Top](../index.html)

#### InterpreterRuntime クラス関連のクラス (InterpreterRuntime, SignatureHandlerLibrary, 及びそれらの補助クラス(UnlockFlagSaver))

これらは, Interpreter 用のランタイム機能を提供するクラス (See: [here](no7882AgC.html) for details).


### クラス一覧(class list)

  * [InterpreterRuntime](#noGevbpCNu)
  * [SignatureHandlerLibrary](#noDW-9kr3H)
  * [UnlockFlagSaver](#no7pOOp36P)


---
## <a name="noGevbpCNu" id="noGevbpCNu">InterpreterRuntime</a>

### 概要(Summary)
HotSpot 内にある Runtime クラスの1つ 
(= アセンブリでは書くことが難しい複雑な機能(例外ハンドリング, 重量ロック処理, etc)を納めた名前空間(AllStatic クラス))
(See: [here](no1904gX2.html) for details).

その中でも InterpreterRuntime クラスには (名前の通り) Interpreter から呼び出されるルーチンが納められている.


```
    ((cite: hotspot/src/share/vm/interpreter/interpreterRuntime.hpp))
    // The InterpreterRuntime is called by the interpreter for everything
    // that cannot/should not be dealt with in assembly and needs C support.
    
    class InterpreterRuntime: AllStatic {
```

### 使われ方(Usage)
Interpreter 関連の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
例えば, ldc や new, 例外送出(throw_*), resolve 処理, monitorenter/monitorexit 処理, 等の機能を提供している.


```
    ((cite: hotspot/src/share/vm/interpreter/interpreterRuntime.hpp))
     public:
      // Constants
      static void    ldc           (JavaThread* thread, bool wide);
      static void    resolve_ldc   (JavaThread* thread, Bytecodes::Code bytecode);
    
      // Allocation
      static void    _new          (JavaThread* thread, constantPoolOopDesc* pool, int index);
      static void    newarray      (JavaThread* thread, BasicType type, jint size);
      static void    anewarray     (JavaThread* thread, constantPoolOopDesc* pool, int index, jint size);
      static void    multianewarray(JavaThread* thread, jint* first_size_address);
      static void    register_finalizer(JavaThread* thread, oopDesc* obj);
    
    ...
      // Exceptions thrown by the interpreter
      static void    throw_AbstractMethodError(JavaThread* thread);
      static void    throw_IncompatibleClassChangeError(JavaThread* thread);
      static void    throw_StackOverflowError(JavaThread* thread);
      static void    throw_ArrayIndexOutOfBoundsException(JavaThread* thread, char* name, jint index);
    ...
    
      // Statics & fields
      static void    resolve_get_put(JavaThread* thread, Bytecodes::Code bytecode);
    
      // Synchronization
      static void    monitorenter(JavaThread* thread, BasicObjectLock* elem);
      static void    monitorexit (JavaThread* thread, BasicObjectLock* elem);
    ...
    
      // Calls
      static void    resolve_invoke       (JavaThread* thread, Bytecodes::Code bytecode);
      static void    resolve_invokedynamic(JavaThread* thread);
    ...
```




### 詳細(Details)
See: [here](../doxygen/classInterpreterRuntime.html) for details

---
## <a name="noDW-9kr3H" id="noDW-9kr3H">SignatureHandlerLibrary</a>

### 概要(Summary)
InterpreterRuntime クラス内で使用される補助クラス.

一度作ったシグネチャハンドラ(signature handler) をメモイズしておくためのクラス 
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)). (See: [here](no3059asZ.html) for details)

(なおシグネチャハンドラとは, 
ネイティブメソッドの呼び出し時にオペランドスタックの引数を ABI にしたがった形に変換する関数のこと.
それぞれのネイティブメソッドに対し, 引数の型や個数に応じて適切なシグネチャハンドラが実行時に生成される.
ただし, 引数の個数/型が共通のネイティブメソッド同士では流用できるのでこのクラスでメモイズしている)


```
    ((cite: hotspot/src/share/vm/interpreter/interpreterRuntime.hpp))
    class SignatureHandlerLibrary: public AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classSignatureHandlerLibrary.html) for details

---
## <a name="no7pOOp36P" id="no7pOOp36P">UnlockFlagSaver</a>

### 概要(Summary)
InterpreterRuntime クラス内で使用される補助クラス.

ソースコード中のあるスコープの間だけ, JavaThread の _do_not_unlock フィールドを false にするための一時オブジェクト.


```
    ((cite: hotspot/src/share/vm/interpreter/interpreterRuntime.cpp))
    class UnlockFlagSaver {
```

### 内部構造(Internal structure)
コンストラクタで (古い _do_not_unlock を待避した後で) false に設定し, デストラクタで元の値に戻している.


```
    ((cite: hotspot/src/share/vm/interpreter/interpreterRuntime.cpp))
        UnlockFlagSaver(JavaThread* t) {
          _thread = t;
          _do_not_unlock = t->do_not_unlock_if_synchronized();
          t->set_do_not_unlock_if_synchronized(false);
        }
        ~UnlockFlagSaver() {
          _thread->set_do_not_unlock_if_synchronized(_do_not_unlock);
        }
```




### 詳細(Details)
See: [here](../doxygen/classUnlockFlagSaver.html) for details

---
