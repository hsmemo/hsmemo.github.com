---
layout: default
title: ServiceUtil クラス 
---
[Top](../index.html)

#### ServiceUtil クラス 



---
## <a name="nomwcEUCRj" id="nomwcEUCRj">ServiceUtil</a>

### 概要(Summary)
保守運用機能のためのクラス (JVMTI 機能及び JMM 機能用のクラス).

ServiceUtil::visible_oop() というユーティリティ・メソッド(のみ)を納めた名前空間(AllStatic クラス)
(これ以外にメソッドやフィールドはない).

```
    ((cite: hotspot/src/share/vm/services/serviceUtil.hpp))
    //
    // Serviceability utility functions.
    // (Shared by MM and JVMTI).
    //
    class ServiceUtil : public AllStatic {
```

ServiceUtil::visible_oop() は, 
引数で指定された oop が「Java のプログラムからも見えるかどうか」を返す関数.

(例えば, Java のインスタンスオブジェクトやクラスオブジェクト, 配列といった oop であれば true が返される.
 逆に deleted_handle や klassOop や methodOop や ... といった oop の場合は false が返される.)


```
    ((cite: hotspot/src/share/vm/services/serviceUtil.hpp))
      // Return true if oop represents an object that is "visible"
      // to the java world.
```

#### 参考(for your information): ServiceUtil::visible_oop()
See: [here](no5248Pap.html) for details
### 使われ方(Usage)
(例えば,
 JVMTI の "monitor contended enter" でイベントを送信することになっている場合に,
 モニターのロックを握っているのが HotSpot の内部的なオブジェクトであれば(通知してもしょうが無いので)通知しない, といった判定処理で使われている模様.)




### 詳細(Details)
See: [here](../doxygen/classServiceUtil.html) for details

---
