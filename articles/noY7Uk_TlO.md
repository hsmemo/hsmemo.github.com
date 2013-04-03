---
layout: default
title: klassOopDesc クラス 
---
[Top](../index.html)

#### klassOopDesc クラス 



---
## <a name="noA9KDmabh" id="noA9KDmabh">klassOopDesc</a>

### 概要(Summary)
oopDesc オブジェクトの種別(「クラス」)を表すためのクラス
(= 各 oopDesc クラスに対する「クラスオブジェクト」的な存在).

なお, klassOopDesc 自身には「クラス」を表す情報は含まれていない.
代わりに klassOopDesc 内に格納している Klass クラス (のサブクラス) のオブジェクトがその情報を保持している
(C++ レベルでのクラスに応じたダイナミックディスパッチ処理は Klass が提供している).

このため klassOopDesc オブジェクトは常に Klass オブジェクトとセットで使用される.
より正確に言うと Klass オブジェクトは KlassOop の中に埋め込まれている.
このため, これらのための領域は KlassOop 分が一度に確保され, その中の一部を Klass オブジェクトとして使う, という感じになる
(See: Klass).


```
    ((cite: hotspot/src/share/vm/oops/klassOop.hpp))
    // A klassOop is the C++ equivalent of a Java class.
    // Part of a klassOopDesc is a Klass which handle the
    // dispatching for the C++ method calls.
```


```
    ((cite: hotspot/src/share/vm/oops/klassOop.hpp))
    class klassOopDesc : public oopDesc {
```

### 使われ方(Usage)
(Klass オブジェクトとセットで使われるため, 各 Klass オブジェクトの使われ方を参照)

(また oopDesc の 2 word 目に埋め込まれるため, oopDesc も参照)

### 内部構造(Internal structure)
メモリ上には Klass とセットで確保される (Klass は常に klassOopDesc の一部として存在する) (See: Klass).

メモリ上でのレイアウトは以下の通り.

  * header :
    oopDesc の部分 (mark と klass)
  * klass_field :
    (フィールドを宣言してないがここにもデータがある模様)
  * KLASS :
    (フィールドを宣言してないがここに Klass のデータがある.
    klassOopDesc::klass_part() でアクセスできる (あるいはそのラッパーである Klass::cast() でもいい))


```
    ((cite: hotspot/src/share/vm/oops/klassOop.hpp))
    //  klassOop object layout:
    //    [header     ]
    //    [klass_field]
    //    [KLASS      ]
```

### 備考(Notes)
なお, 実際の使用箇所では klassOop という別名(もしくはラッパークラス)で使われることが多い (See: klassOop).

### 備考(Notes)
klassOopDesc は Java の「クラスオブジェクト (HotSpot 用語では "mirror オブジェクト")」を表すものではない.
クラスオブジェクトは instanceOopDesc で表現される.

(ただしクラスオブジェクトの場合は普通の instanceOopDesc とは少し異なる.
より具体的に言うと, クラスオブジェクトの場合にのみ対応する klassOopDesc の Klass が instanceMirrorKlass になる)


### 備考(Notes)
klassOopDesc から Klass を取得するには,
Klass::cast() (または, Klass::cast() が内側で呼んでいる klassOopDesc::klass_part()) を使えばいい.


```
    ((cite: hotspot/src/share/vm/oops/klass.hpp))
      // Casting
      static Klass* cast(klassOop k) {
        assert(k->is_klass(), "cast to Klass");
        return k->klass_part();
      }
```




### 詳細(Details)
See: [here](../doxygen/classklassOopDesc.html) for details

---
