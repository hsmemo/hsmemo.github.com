---
layout: default
title: LinkResolver クラス関連のクラス (LinkInfo, FieldAccessInfo, CallInfo, LinkResolver)
---
[Top](../index.html)

#### LinkResolver クラス関連のクラス (LinkInfo, FieldAccessInfo, CallInfo, LinkResolver)

これらは, ConstantPool の解決処理 (resolve 処理 = ダイナミックリンク処理) を行うためのクラス (See: [here](no7882NqI.html) for details).

(HotSpot 内では, クラス, フィールド, メソッドなどのシンボル名は最初参照された際にダイナミックリンク(resolve)される)


```
    ((cite: hotspot/src/share/vm/interpreter/linkResolver.hpp))
    // All the necessary definitions for run-time link resolution.
```



### クラス一覧(class list)

  * [LinkResolver](#nobJeQAXWL)
  * [LinkInfo](#no4VJIUNES)
  * [FieldAccessInfo](#nojJ5EelTC)
  * [CallInfo](#noz58ZWb82)


---
## <a name="nobJeQAXWL" id="nobJeQAXWL">LinkResolver</a>

ConstantPool の解決処理 (resolve 処理) を行うクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```
    ((cite: hotspot/src/share/vm/interpreter/linkResolver.hpp))
    // The LinkResolver is used to resolve constant-pool references at run-time.
    // It does all necessary link-time checks & throws exceptions if necessary.
    
    class LinkResolver: AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classLinkResolver.html) for details

---
## <a name="no4VJIUNES" id="no4VJIUNES">LinkInfo</a>

LinkResolver クラス用の補助クラス(の基底クラス).

resolve 処理で使用されるリンク情報を入れておくためのクラス
(e.g. フィールド名, メソッド名, クラス名, etc).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/interpreter/linkResolver.hpp))
    // LinkInfo & its subclasses provide all the information gathered
    // for a particular link after resolving it. A link is any reference
    // made from within the bytecodes of a method to an object outside of
    // that method. If the info is invalid, the link has not been resolved
    // successfully.
```


```
    ((cite: hotspot/src/share/vm/interpreter/linkResolver.hpp))
    class LinkInfo VALUE_OBJ_CLASS_SPEC {
    };
```

### 内部構造(Internal structure)
LinkInfo クラス自体には, 何のフィールドもメソッドも定義されていない.




### 詳細(Details)
See: [here](../doxygen/classLinkInfo.html) for details

---
## <a name="nojJ5EelTC" id="nojJ5EelTC">FieldAccessInfo</a>

LinkInfo クラスの具象サブクラスの1つ.

フィールドアクセス (getfield/putfield & getstatic/putstatic) に関するリンク情報を格納するクラス.


```
    ((cite: hotspot/src/share/vm/interpreter/linkResolver.hpp))
    // Link information for getfield/putfield & getstatic/putstatic bytecodes.
    
    class FieldAccessInfo: public LinkInfo {
```




### 詳細(Details)
See: [here](../doxygen/classFieldAccessInfo.html) for details

---
## <a name="noz58ZWb82" id="noz58ZWb82">CallInfo</a>

LinkInfo クラスの具象サブクラスの1つ.

メソッド呼び出し (invoke*) に関するリンク情報を格納するクラス.


```
    ((cite: hotspot/src/share/vm/interpreter/linkResolver.hpp))
    // Link information for all calls.
    
    class CallInfo: public LinkInfo {
```




### 詳細(Details)
See: [here](../doxygen/classCallInfo.html) for details

---
