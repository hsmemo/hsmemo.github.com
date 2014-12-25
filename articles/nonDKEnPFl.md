---
layout: default
title: ArchDesc クラス関連のクラス (ChainList, MatchList, ArchDesc, OutputMap)
---
[Top](../index.html)

#### ArchDesc クラス関連のクラス (ChainList, MatchList, ArchDesc, OutputMap)



### クラス一覧(class list)

  * [ChainList](#no4Q0RWD2a)
  * [MatchList](#no9bCvfuzT)
  * [ArchDesc](#novdQryxPz)
  * [OutputMap](#nopJzqqLVu)


---
## <a name="no4Q0RWD2a" id="no4Q0RWD2a">ChainList</a>

### 概要(Summary)
ADL ファイル中の OPERANDS 節 (INSTRUCTIONS 節 ?) の chain match rule を記憶しておくためのクラス(? #TODO). (See: [here](nop0Yyr-jc.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/adlc/archDesc.hpp))
    class ChainList {
```



### 詳細(Details)
See: [here](../doxygen/classChainList.html) for details

---
## <a name="no9bCvfuzT" id="no9bCvfuzT">MatchList</a>

### 概要(Summary)
ADL ファイル中の OPERANDS 節 (INSTRUCTIONS 節 ?) の match rule を記憶しておくためのクラス(? #TODO). (See: [here](nop0Yyr-jc.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/adlc/archDesc.hpp))
    class MatchList {
```



### 詳細(Details)
See: [here](../doxygen/classMatchList.html) for details

---
## <a name="novdQryxPz" id="novdQryxPz">ArchDesc</a>

### 概要(Summary)
ADL ファイルのパース結果を格納し, match rule 等から DFA 等を構築するクラス. (See: [here](nop0Yyr-jc.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/adlc/archDesc.hpp))
    class ArchDesc {
```



### 詳細(Details)
See: [here](../doxygen/classArchDesc.html) for details

---
## <a name="nopJzqqLVu" id="nopJzqqLVu">OutputMap</a>

### 概要(Summary)
ADLC の処理結果を cpp/hpp ファイルに書き出すための補助クラス (? #TODO). (See: [here](nop0Yyr-jc.html) for details)

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/adlc/archDesc.hpp))
    // Base class for generating a mapping from rule number to value.
    // Used with ArchDesc::build_map() for all maps except "enum MachOperands"
    // A derived class defines the appropriate output for a specific mapping.
    class OutputMap {
```




### 詳細(Details)
See: [here](../doxygen/classOutputMap.html) for details

---
