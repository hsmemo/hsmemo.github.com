---
layout: default
title: JIT Compiler ： 補足 ： C2 JIT コンパイラ用の前処理 ： 概要
---
[Up](noVxQtU9lk.html) [Top](../index.html)

#### JIT Compiler ： 補足 ： C2 JIT コンパイラ用の前処理 ： 概要

--- 
## 概要(Summary)
C2 JIT コンパイラは, コンパイル時に対象のアーキテクチャの情報を必要とする
(例: 利用可能なマシン語・レジスタ, スタックフレームのレイアウト, CPU のパイプライン情報, 等). 
HotSpot のソースコード内では, これらの情報のほとんどは「AD ファイル (または ADL ファイル)」というファイルに書かれている.

AD ファイル自体はソースコードとして直接使える形式にはなっておらず,
まず adlc(ADL Compiler) というプログラムが AD ファイルを C++ コードに変換し,
次にこれを基にして C2 JIT コンパイラが生成される.
このため, HotSpot のコンパイルは以下のような手順で行われる (See: [here](nojB2Sq0ue.html) for details).

  (1) まず adlc 自体を生成する.
      (なお, adlc を構成するソースコードは hotspot/src/share/vm/adlc 以下に存在する. (See: [here](noxdhIMi-n.html) for details))

  (2) adlc が、AD ファイルから C++ ソースコードを生成する. (See: [here](nop0Yyr-jc.html) for details)

  (3) できた C++ コードと残りの HotSpot のソースコードから、HotSpot が生成される.

## 備考(Notes)
なお, ADL は "Architecture Description Language" の略らしい.

## 備考(Notes)
AD ファイルは hotspot/src/cpu/ 下, および hotspot/src/os_cpu 下に置かれている.
  
<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table6348LBc -->
| Arthitecture | AD files |
|---|---|
| Solaris/Sparc | hotspot/src/cpu/sparc/vm/sparc.ad, hotspot/src/os_cpu/solaris_sparc/vm/solaris_sparc.ad (<= ただしこちらは空) |
|  |  |
<!-- END RECEIVE ORGTBL table6348LBc -->

<!-- 
#+ORGTBL: SEND table6348LBc orgtbl-to-gfm :no-escape t
| Arthitecture  | AD files                                                                                                      |
|---------------+---------------------------------------------------------------------------------------------------------------|
| Solaris/Sparc | hotspot/src/cpu/sparc/vm/sparc.ad, hotspot/src/os_cpu/solaris_sparc/vm/solaris_sparc.ad (<= ただしこちらは空) |
|               |                                                                                                               |
-->

## 参考(for your information)
adlc によって AD ファイルから生成されるソースコードファイルは以下の通り. (なお, ${architecture} の箇所にはそれぞれのビルド環境を表す文字列が入る. 例えば dfa_x86_64.cpp, 等)

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table6348NYG -->
| source code file | description |
|---|---|
| dfa_${architecture}.cpp | Matcher クラス(See: [here](nomNJlPgZt.html) for details)の DFA(決定性有限オートマトン)用のルールを規定するソース |
| ad_${architecture}.hpp | AD ファイルから生成される MachNode のサブクラスおよび MachOper のサブクラスの宣言 |
| ad_${architecture}.cpp | AD ファイルから生成される MachNode のサブクラスおよび MachOper のサブクラスの定義 |
| ad_${architecture}_clone.cpp | 〃サブクラスの clone() メソッドの定義 |
| ad_${architecture}_expand.cpp | 〃サブクラスの Expand() メソッドの定義 |
| ad_${architecture}_format.cpp | 〃サブクラスの int_format() メソッド, ext_format() メソッド, format() メソッドの定義 (これらはデバッグ出力用のメソッド) |
| ad_${architecture}_gen.cpp | 〃サブクラスのインスタンスを生成するメソッド (State::MachOperGenerator() および State::MachNodeGenerator()) の定義 |
| ad_${architecture}_misc.cpp | 〃サブクラスのその他のメソッドの定義 |
| ad_${architecture}_peephole.cpp | MachNode サブクラスの peephole() メソッドの定義 (See: PhasePeephole) |
| ad_${architecture}_pipeline.cpp | MachNode サブクラスの pipeline() メソッド, pipeline_class() メソッドの定義 (See: Scheduling) |
| adGlobals_${architecture}.hpp | アーキテクチャ依存の定数の定義 |
<!-- END RECEIVE ORGTBL table6348NYG -->

<!-- 
#+ORGTBL: SEND table6348NYG orgtbl-to-gfm :no-escape t
| source code file                | description                                                                                                             |
|---------------------------------+-------------------------------------------------------------------------------------------------------------------------|
| dfa_${architecture}.cpp         | Matcher クラス(See: [here](nomNJlPgZt.html) for details)の DFA(決定性有限オートマトン)用のルールを規定するソース                               |
| ad_${architecture}.hpp          | AD ファイルから生成される MachNode のサブクラスおよび MachOper のサブクラスの宣言                                       |
| ad_${architecture}.cpp          | AD ファイルから生成される MachNode のサブクラスおよび MachOper のサブクラスの定義                                       |
| ad_${architecture}_clone.cpp    | 〃サブクラスの clone() メソッドの定義                                                                                   |
| ad_${architecture}_expand.cpp   | 〃サブクラスの Expand() メソッドの定義                                                                                  |
| ad_${architecture}_format.cpp   | 〃サブクラスの int_format() メソッド, ext_format() メソッド, format() メソッドの定義 (これらはデバッグ出力用のメソッド) |
| ad_${architecture}_gen.cpp      | 〃サブクラスのインスタンスを生成するメソッド (State::MachOperGenerator() および State::MachNodeGenerator()) の定義      |
| ad_${architecture}_misc.cpp     | 〃サブクラスのその他のメソッドの定義                                                                                    |
| ad_${architecture}_peephole.cpp | MachNode サブクラスの peephole() メソッドの定義 (See: PhasePeephole)                                                    |
| ad_${architecture}_pipeline.cpp | MachNode サブクラスの pipeline() メソッド, pipeline_class() メソッドの定義 (See: Scheduling)                            |
| adGlobals_${architecture}.hpp   | アーキテクチャ依存の定数の定義                                                                                          |
-->


```cpp
    ((cite: hotspot/src/share/vm/adlc/archDesc.hpp))
      // Machine dependent files, built from architecture definition
      ADLFILE  _DFA_file;          // File for definition of Matcher::DFA
      ADLFILE  _HPP_file;          // File for ArchNode class declarations
      ADLFILE  _CPP_file;          // File for ArchNode class defintions
      ADLFILE  _CPP_CLONE_file;    // File for MachNode/Oper clone defintions
      ADLFILE  _CPP_EXPAND_file;   // File for MachNode expand methods
      ADLFILE  _CPP_FORMAT_file;   // File for MachNode/Oper format defintions
      ADLFILE  _CPP_GEN_file;      // File for MachNode/Oper generator methods
      ADLFILE  _CPP_MISC_file;     // File for miscellaneous MachNode/Oper tables & methods
      ADLFILE  _CPP_PEEPHOLE_file; // File for MachNode peephole methods
      ADLFILE  _CPP_PIPELINE_file; // File for MachNode pipeline defintions
      ADLFILE  _VM_file;           // File for constants needed in VM code
      ADLFILE  _bug_file;          // DFA debugging file
```

```cpp
    ((cite: hotspot/src/share/vm/adlc/main.cpp))
          const char *base = strip_ext(strdup(argv[i]));
          char       *temp = base_plus_suffix("dfa_",base);
          AD._DFA_file._name = base_plus_suffix(temp,".cpp");
          delete temp;
          temp = base_plus_suffix("ad_",base);
          AD._CPP_file._name          = base_plus_suffix(temp,".cpp");
          AD._CPP_CLONE_file._name    = base_plus_suffix(temp,"_clone.cpp");
          AD._CPP_EXPAND_file._name   = base_plus_suffix(temp,"_expand.cpp");
          AD._CPP_FORMAT_file._name   = base_plus_suffix(temp,"_format.cpp");
          AD._CPP_GEN_file._name      = base_plus_suffix(temp,"_gen.cpp");
          AD._CPP_MISC_file._name     = base_plus_suffix(temp,"_misc.cpp");
          AD._CPP_PEEPHOLE_file._name = base_plus_suffix(temp,"_peephole.cpp");
          AD._CPP_PIPELINE_file._name = base_plus_suffix(temp,"_pipeline.cpp");
          AD._HPP_file._name = base_plus_suffix(temp,".hpp");
          delete temp;
          temp = base_plus_suffix("adGlobals_",base);
          AD._VM_file._name = base_plus_suffix(temp,".hpp");
          delete temp;
          temp = base_plus_suffix("bugs_",base);
          AD._bug_file._name = base_plus_suffix(temp,".out");
          delete temp;
```







