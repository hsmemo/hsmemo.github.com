---
layout: default
title: JvmtiClassFileReconstituter クラス関連のクラス (JvmtiConstantPoolReconstituter, JvmtiClassFileReconstituter)
---
[Top](../index.html)

#### JvmtiClassFileReconstituter クラス関連のクラス (JvmtiConstantPoolReconstituter, JvmtiClassFileReconstituter)

これらは, JVMTI の関数を実装するために使われているクラス.
より具体的に言うと, GetConstantPool() 関数, GetBytecodes() 関数, および RetransformClasses() 関数を実装するためのクラス
(See: [here](no2935xEp.html), [here](no2935Lnd.html) and [here](no2935-Vj.html) for details).

### 概要(Summary)
これらのクラスは, 指定されたクラスの constant pool 情報や class file 全体の情報(あるいはその中の bytecode 情報)が必要となった際に,
HotSpot 内部のデータ構造から元の(JVMSで規定されている形式の)情報を復元する処理を行う.

JvmtiConstantPoolReconstituter と JvmtiClassFileReconstituter はそれぞれ
Constant Pool とクラスファイル全体(あるいはその一部)の復元を担当する.

なお, RetransformClasses() で JvmtiClassFileReconstituter が使われているのは,
RetransformClasses() で指定されたロード済みクラスのクラスファイル情報を ClassFileLoadHook イベントのコールバックに渡す必要があるため.


### クラス一覧(class list)

  * [JvmtiConstantPoolReconstituter](#noChndGVhE)
  * [JvmtiClassFileReconstituter](#noSnustF7j)


---
## <a name="noChndGVhE" id="noChndGVhE">JvmtiConstantPoolReconstituter</a>

### 概要(Summary)
指定されたクラスの Constant Pool 情報を 
JVMS で規定された形式に復元するための一時オブジェクト(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiClassFileReconstituter.hpp))
    class JvmtiConstantPoolReconstituter : public StackObj {
```

### 使われ方(Usage)
JvmtiEnv::GetConstantPool() 内で(のみ)使用されている (See: [here](no2935xEp.html) for details).




### 詳細(Details)
See: [here](../doxygen/classJvmtiConstantPoolReconstituter.html) for details

---
## <a name="noSnustF7j" id="noSnustF7j">JvmtiClassFileReconstituter</a>

### 概要(Summary)
JvmtiConstantPoolReconstituter クラスのサブクラス.

JvmtiConstantPoolReconstituter との違いは Constant Pool 情報だけでなくクラスファイル全体の復元を行う点.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiClassFileReconstituter.hpp))
    class JvmtiClassFileReconstituter : public JvmtiConstantPoolReconstituter {
```

### 使われ方(Usage)
JvmtiEnv::GetBytecodes() と JvmtiEnv::RetransformClasses() 内で(のみ)使用されている (See: [here](no2935Lnd.html) and [here](no2935-Vj.html) for details)

(なお, JvmtiEnv::GetBytecodes() の中では JvmtiClassFileReconstituter::copy_bytecodes() が呼び出されているだけ)




### 詳細(Details)
See: [here](../doxygen/classJvmtiClassFileReconstituter.html) for details

---
