---
layout: default
title: ADLParser クラス 
---
[Top](../index.html)

#### ADLParser クラス 



---
## <a name="noJh_fqOdn" id="noJh_fqOdn">ADLParser</a>

### 概要(Summary)
ADL ファイルのパース処理を行うクラス (See: [here](nop0Yyr-jc.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/adlc/adlparse.hpp))
    class ADLParser {
```

### 使われ方(Usage)
adlc の main() 関数の中で作成され, 
parse() メソッドの呼び出しが終われば破棄される.


```cpp
    ((cite: hotspot/src/share/vm/adlc/main.cpp))
    int main(int argc, char *argv[])
    {
    ...
      ADLParser    *ADL_Parse;      // ADL Parser object to parse AD file
    ...
      ADL_Parse = new ADLParser(ADL_Buf, AD); // Create a parser to parse the buffer
      ADL_Parse->parse();           // Parse buffer & build description lists
    
    ...
      delete ADL_Parse;             // Delete parser
```




### 詳細(Details)
See: [here](../doxygen/classADLParser.html) for details

---
