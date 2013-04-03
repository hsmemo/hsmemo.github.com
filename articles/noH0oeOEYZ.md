---
layout: default
title: PcDesc クラス 
---
[Top](../index.html)

#### PcDesc クラス 



---
## <a name="noXQURoWtD" id="noXQURoWtD">PcDesc</a>

### 概要(Summary)
JIT コンパイラが生成したコードについて, そのマシン語のアドレスから元の bytecode を逆引きするためのクラス.


```
    ((cite: hotspot/src/share/vm/code/pcDesc.hpp))
    // PcDescs map a physical PC (given as offset from start of nmethod) to
    // the corresponding source scope and byte code index.
```


```
    ((cite: hotspot/src/share/vm/code/pcDesc.hpp))
    class PcDesc VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PcDesc は, nmethod オブジェクト内に埋め込まれている (See: nmethod). (他にも生成されている箇所はある?? #TODO)

nmethod から取り出すには nmethod::pc_desc_at() を使う
(マシン語命令のメモリ上のアドレスを渡すと, 対応する PcDesc が返される).
(他にも取得している箇所はある?? #TODO)


```
    ((cite: hotspot/src/share/vm/code/nmethod.hpp))
      // ScopeDesc retrieval operation
      PcDesc* pc_desc_at(address pc)   { return find_pc_desc(pc, false); }
```




### 詳細(Details)
See: [here](../doxygen/classPcDesc.html) for details

---
