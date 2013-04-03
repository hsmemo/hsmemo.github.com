---
layout: default
title: Parse::do_one_bytecode() 用の補助クラス (SwitchRange) 
---
[Top](../index.html)

#### Parse::do_one_bytecode() 用の補助クラス (SwitchRange) 



---
## <a name="noHFcmPqcG" id="noHFcmPqcG">SwitchRange</a>

### 概要(Summary)
Parse クラス内で使用される補助クラス.

lookupswitch バイトコード及び tableswitch バイトコードの飛び先情報を表すクラス.
1つの SwitchRange オブジェクトが 1つの飛び先情報に対応する.

(なお, このクラスは StackObj クラスだが局所変数としては生成されていない (NEW_RESOURCE_ARRAY() でのみ確保されている))


```
    ((cite: hotspot/src/share/vm/opto/parse2.cpp))
    class SwitchRange : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* Parse::do_tableswitch() 
* Parse::do_lookupswitch()

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* jint _lo, jint _hi
  
  マッチする数値の下限／上限 (値がこの範囲にあれば, この飛び先にマッチする)

* int _dest
  
  マッチした場合の飛び先

* int _table_index
  
  lookupswitch/tableswitch バイトコード中での順番


```
    ((cite: hotspot/src/share/vm/opto/parse2.cpp))
      // a range of integers coupled with a bci destination
      jint _lo;                     // inclusive lower limit
      jint _hi;                     // inclusive upper limit
      int _dest;
      int _table_index;             // index into method data table
```





### 詳細(Details)
See: [here](../doxygen/classSwitchRange.html) for details

---
