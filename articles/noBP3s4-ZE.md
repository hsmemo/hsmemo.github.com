---
layout: default
title: JavaAssertions およびその補助クラス (JavaAssertions, JavaAssertions::OptionList)
---
[Top](../index.html)

#### JavaAssertions およびその補助クラス (JavaAssertions, JavaAssertions::OptionList)

これらは, Java の Assertion 機能を実現するためのクラス (See: [here](no7882m9N.html) for details).

(より正確に言うと, これらはコマンドラインオプションで指定された Assertion 関連の設定値を覚えておくためのクラス.
この情報が java.lang.Class.desiredAssertionStatus() を実現するために使用される)



### クラス一覧(class list)

  * [JavaAssertions](#noLOxxXMgO)
  * [JavaAssertions::OptionList](#nobfhRsLfA)


---
## <a name="noLOxxXMgO" id="noLOxxXMgO">JavaAssertions</a>

### 概要(Summary)
Assertion の設定値管理に関する関数を納めた名前空間(AllStatic クラス)


```
    ((cite: hotspot/src/share/vm/classfile/javaAssertions.hpp))
    class JavaAssertions: AllStatic {
```

### 使われ方(Usage)
* コマンドラインオプションのパース時に,
  JavaAssertions のメソッドが呼び出されて設定値が格納される (See: [here](no7882m9N.html) for details).

* 蓄えた情報は JVM_DesiredAssertionStatus() や JVM_AssertionStatusDirectives() で参照される
  (これらは java.lang.Class.desiredAssertionStatus() から呼び出される補助関数).


```
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    JVM_ENTRY(jboolean, JVM_DesiredAssertionStatus(JNIEnv *env, jclass unused, jclass cls))
    ...
      return JavaAssertions::enabled(name, system_class);
    
    JVM_END
    
    
    // Return a new AssertionStatusDirectives object with the fields filled in with
    // command-line assertion arguments (i.e., -ea, -da).
    JVM_ENTRY(jobject, JVM_AssertionStatusDirectives(JNIEnv *env, jclass unused))
    ...
      oop asd = JavaAssertions::createAssertionStatusDirectives(CHECK_NULL);
      return JNIHandles::make_local(env, asd);
    JVM_END
```




### 詳細(Details)
See: [here](../doxygen/classJavaAssertions.html) for details

---
## <a name="nobfhRsLfA" id="nobfhRsLfA">JavaAssertions::OptionList</a>

### 概要(Summary)
JavaAssertions クラス内で使用される補助クラス.

指定されたオプション値を格納しておくための線形リスト.


```
    ((cite: hotspot/src/share/vm/classfile/javaAssertions.hpp))
    class JavaAssertions::OptionList: public CHeapObj {
```

### 内部構造(Internal structure)
定義されているフィールドはこれだけ
(そして, メソッドはこれらのフィールドへの getter メソッド(アクセサメソッド)のみ).


```
    ((cite: hotspot/src/share/vm/classfile/javaAssertions.hpp))
      const char*   _name;
      OptionList*   _next;
      bool          _enabled;
```




### 詳細(Details)
See: [here](../doxygen/classJavaAssertions_1_1OptionList.html) for details

---
