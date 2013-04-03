---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下
---
[Up](nooSr9ojgi.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下

--- 

File Name                                         | Description
------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/                                | OS/CPU に非依存のソースコード用のディレクトリ
- hotspot/src/share/tools/                        | HotSpot の開発時に役立つツール類のソースコードを納めたディレクトリ
- - hotspot/src/share/tools/IdealGraphVisualizer/ | C2 JIT Compiler が扱う高レベル中間語("Ideal")のグラフをGUIで表示するためのツールのソースコード
- - hotspot/src/share/tools/LogCompilation/       | -XX:+LogCompilationオプション(コンパイルの作業内容をログに出力)の出力結果を可視化するツールのソースコード
- - hotspot/src/share/tools/ProjectCreator/       | HotSpot のソースコード用に IDE のプロジェクトを作るツールのソースコード
- - hotspot/src/share/tools/hsdis/                | HotSpot のデバッグ用機能(-XX:+PrintAssembly)で使用される逆アセンブラのソースコード
- - hotspot/src/share/tools/launcher/             | gamma(テスト用の簡易的な java コマンド) のソースコード
- hotspot/src/share/vm/                           | HotSpot 本体のソースコード用のディレクトリ
- - hotspot/src/share/vm/adlc/                    | C2 JIT Compiler 用のプラットフォーム依存なソースコードを生成するコンパイラ("ADLC")のソースコード
- - hotspot/src/share/vm/asm/                     | 動的コード生成を補佐するクラス(Assembler)関連のソースコード(より正確にはそれらのプラットフォーム非依存な部分のソースコード) (AbstractAssembler, CodeBuffer, AbstractRegisterImpl, etc)
- - hotspot/src/share/vm/c1/                      | C1 JIT Compiler 関連のソースコード
- - hotspot/src/share/vm/ci/                      | JIT Compiler とその他の部分をつなぐインターフェース("Compiler Interface")のソースコード
- - hotspot/src/share/vm/classfile/               | クラスファイル管理(クラスローディング処理も含む)に関するソースコード (ClassFileParser, ClassLoader, SystemDictionary, StackMapTable, etc)
- - hotspot/src/share/vm/code/                    | 動的生成コードを格納するためのメモリ領域(CodeCache)関連のソースコード (CodeCache, ICBuffer, nmethod, relocInfo, etc)
- - hotspot/src/share/vm/compiler/                | JIT Compiler の管理用スレッド関連のソースコード (AbstractCompiler, CompilerBroker, oopMap, etc)
- - hotspot/src/share/vm/gc_implementation/       | 各種 GC アルゴリズムのソースコード (ParNew, CMS, ParallelScavenge, G1GC)
- - hotspot/src/share/vm/gc_interface/            | GC アルゴリズム用の共通インターフェースのソースコード (CollectedHeap, GCCause, etc)
- - hotspot/src/share/vm/interpreter/             | インタープリタ関連のソースコード (abstractInterpreter, cppInterpreter, interpreterGenerator, interpreterRuntime, templateInterpreter, etc)
- - hotspot/src/share/vm/libadt/                  | ADLC(上述) が使用するデータ構造(ADT, Abstract Data Type)に関するソースコード
- - hotspot/src/share/vm/memory/                  | Javaヒープ管理に関するソースコード (barrierSet, cardTableRS, collectorPolicy, universe, generation, defNewGeneration, etc)
- - hotspot/src/share/vm/oops/                    | Java のオブジェクト管理 (HotSpot 内での Java オブジェクトの内部表現) に関するソースコード (oop, klass, methodOop, objArrayKlass, symbolKlass, etc)
- - hotspot/src/share/vm/opto/                    | C2 JIT Compiler 関連のソースコード
- - hotspot/src/share/vm/prims/                   | JNI や JVMTI, sun.misc.unsafe 等に関するソースコード
- - hotspot/src/share/vm/runtime/                 | HotSpot のランタイム機能に関するソースコード (SharedRuntime, スレッド管理(OSThread, safepoint, vmThread, etc), 同期排他処理(objectMonitor 等), etc)
- - hotspot/src/share/vm/services/                | HotSpot の保守運用機能に関するソースコード (JMM (Monitoring and Management Interface), Dynamic Attach API, DTrace, etc)
- - hotspot/src/share/vm/shark/                   | Shark JIT Compiler 関連のソースコード
- - hotspot/src/share/vm/utilities/               | HotSpot 内で広く使われるユーティリティクラスや定数定義等のソースコード (globalDefinitions, copy, etc)




## Subcategories
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： adlc/](noxdhIMi-n.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： asm/](noAn29AiCz.html)
* [(#TBD) ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： c1/](noetNHIcFU.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： ci/](nohfYmWRnl.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： classfile/](noPO35aB62.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： code/](noMzP3wBQ4.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： compiler/](noUW150NxH.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： gc_implementation/](noBI-e1EXt.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： gc_interface/](nomTzCJdJn.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： interpreter/](novFbH8Jzy.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： libadt/](nop6s8ZTqe.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： memory/](nod6wZ_uC-.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： oops/](no5SiZDTv-.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： opto/](noSoVw517l.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： prims/](nooQCM7QVB.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： runtime/](nox4jBrGc_.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： services/](no_X23NBjM.html)
* [(#TBD) ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： shark/](noXIWW8NwJ.html)
* [ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： utilities/](noLagopc3K.html)



