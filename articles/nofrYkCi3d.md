---
layout: default
title: StubRoutines クラス用の補助クラス 
---
[Top](../index.html)

#### StubRoutines クラス用の補助クラス 



---
## <a name="no8_fK8wyx" id="no8_fK8wyx">StubRoutines::x86</a>

### 概要(Summary)
StubRoutines クラス用の補助クラス.
x86 プラットフォームに特化したスタブを納めた名前空間(このクラスは AllStatic ではないが, static なフィールド／メソッドしか持たない).

(なお, 32bit か 64bit かによってクラス定義が別になっている.)


```
    ((cite: hotspot/src/cpu/x86/vm/stubRoutines_x86_32.hpp))
    class x86 {
```


```
    ((cite: hotspot/src/cpu/x86/vm/stubRoutines_x86_64.hpp))
    class x86 {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り
(そして, メソッドはこれらのフィールドへの getter メソッド(アクセサメソッド)のみ).

(これらのフィールドには対応するスタブの先頭アドレスが格納される)

* 32bit の場合:

```
    ((cite: hotspot/src/cpu/x86/vm/stubRoutines_x86_32.hpp))
      static address _verify_mxcsr_entry;
      static address _verify_fpu_cntrl_wrd_entry;
```

* 64bit の場合:

```
    ((cite: hotspot/src/cpu/x86/vm/stubRoutines_x86_64.hpp))
      static address _get_previous_fp_entry;
      static address _verify_mxcsr_entry;
    
      static address _f2i_fixup;
      static address _f2l_fixup;
      static address _d2i_fixup;
      static address _d2l_fixup;
    
      static address _float_sign_mask;
      static address _float_sign_flip;
      static address _double_sign_mask;
      static address _double_sign_flip;
      static address _mxcsr_std;
```




### 詳細(Details)
See: [here](../doxygen/classStubRoutines_1_1x86.html) for details

---
