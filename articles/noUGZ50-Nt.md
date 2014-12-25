---
layout: default
title: IdealGraphPrinter クラス 
---
[Top](../index.html)

#### IdealGraphPrinter クラス 



---
## <a name="noHE0dfXjQ" id="noHE0dfXjQ">IdealGraphPrinter</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか使用されない).

デバッグ用(開発時用)のツールである IdealGraphVisualizer への出力を生成するクラス (See: IdealGraphVisualizer).


```cpp
    ((cite: hotspot/src/share/vm/opto/idealGraphPrinter.hpp))
    #ifndef PRODUCT
    ...
    class IdealGraphPrinter
    {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CompilerThread オブジェクトの _ideal_graph_printer フィールドに(のみ)格納されている
(アクセサは IdealGraphPrinter::printer()).

(ただし, IdealGraphPrinter オブジェクトの生成自体は実際に必要になるまで遅延されている)

#### 生成箇所(where its instances are created)
IdealGraphPrinter::printer() 内で(のみ)生成されている (= 初めて使用される時まで生成が遅延されている).

#### 使用箇所(where its instances are used)
コンパイル時に Compile オブジェクトのコンストラクタに渡され, コンパイルの各段階で呼び出されている模様
(Compile オブジェクト内では Compile::_printer フィールドに格納されている).

* コンパイル開始時の begin_method() から呼び出される.
  
  ファイルを開く等の前準備が行われる.

* コンパイル途中の各段階の print_method() から呼び出される.
  
  ここで実際の Ideal が出力される (?? #TODO)

* インライン展開を行った直後 (Finish_Warm() の直後) にも呼び出される.
  
* コンパイル終了時の end_method() から呼び出される.
  
  出力をフラッシュする等の後片付けが行われる.

* ... (#TODO)




### 詳細(Details)
See: [here](../doxygen/classIdealGraphPrinter.html) for details

---
