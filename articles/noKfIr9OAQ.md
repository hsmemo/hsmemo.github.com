---
layout: default
title: PhaseMacroExpand クラス 
---
[Top](../index.html)

#### PhaseMacroExpand クラス 



---
## <a name="noOjWoiBkX" id="noOjWoiBkX">PhaseMacroExpand</a>

### 概要(Summary)
Phase クラスの具象サブクラスの1つ.

メモリ確保処理(AllocateNode/AllocateArrayNode)や同期排他処理(LockNode/UnlockNode)を,
対応するランタイム呼び出し(CallNode)を使った形式に展開する.


```cpp
    ((cite: hotspot/src/share/vm/opto/macro.hpp))
    class PhaseMacroExpand : public Phase {
```

### 使われ方(Usage)
Compile::Optimize() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPhaseMacroExpand.html) for details

---
