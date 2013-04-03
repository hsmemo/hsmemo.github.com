---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： adlc/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： adlc/

--- 

File Name                                          | Description                                                   
-------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/adlc/Doc/                     |  ドキュメント用ディレクトリ
- hotspot/src/share/vm/adlc/Doc/Syntax.doc         |  AD ファイルの syntax に関するドキュメント
hotspot/src/share/vm/adlc/Test/                    |  ??
- hotspot/src/share/vm/adlc/Test/i486.ad           |  ??(空ファイル)
hotspot/src/share/vm/adlc/adlc.hpp                 |  adlc/ ディレクトリ下で共通で使われる関数／定数などの宣言をまとめたヘッダファイル
hotspot/src/share/vm/adlc/adlparse.cpp             |  ADLParser クラスの定義 ([ADLParser](noTY_aBUNo.html))
hotspot/src/share/vm/adlc/adlparse.hpp             |  同上
hotspot/src/share/vm/adlc/archDesc.cpp             |  ArchDesc クラス関連のクラスの定義 ([ChainList, MatchList, ArchDesc, OutputMap](nonDKEnPFl.html))
hotspot/src/share/vm/adlc/archDesc.hpp             |  同上
hotspot/src/share/vm/adlc/arena.cpp                |  ADLC 内でのメモリ管理用のクラスの定義 ([CHeapObj, ValueObj, AllStatic, Chunk, Arena](no2w3qnLDI.html))
hotspot/src/share/vm/adlc/arena.hpp                |  同上
hotspot/src/share/vm/adlc/dfa.cpp                  |  Matcher クラスの DFA を生成する関数(ArchDesc::buildDFA())およびその補助関数/補助クラスの定義 ([Production, ProductionState, dfa_shared_preds](nolxYbkOTh.html))
hotspot/src/share/vm/adlc/dict2.cpp                |  Dict クラス関連のクラスの定義 ([Dict, DictI, 及びそれらの補助クラス(bucket)](no0XbBtCTY.html))
hotspot/src/share/vm/adlc/dict2.hpp                |  同上
hotspot/src/share/vm/adlc/filebuff.cpp             |  FileBuff クラスおよびその補助クラスの定義 ([FileBuff, FileBuffRegion](no476i8Y7T.html))
hotspot/src/share/vm/adlc/filebuff.hpp             |  同上
hotspot/src/share/vm/adlc/forms.cpp                |  Form 関連の基本的なクラスの定義 ([FormDict, Form, FormList, NameList, PreserveIter, NameAndList, ComponentList, SourceForm, HeaderForm, PreHeaderForm, Expr, ExprDict](noST32ewsp.html))
hotspot/src/share/vm/adlc/forms.hpp                |  同上
hotspot/src/share/vm/adlc/formsopt.cpp             |  Form のサブクラスの定義 ([RegisterForm, RegDef, RegClass, AllocClass, FrameForm, PipelineForm, ResourceForm, PipeClassOperandForm, PipeClassResourceForm, PipeClassForm, Peephole, PeepMatch, PeepConstraint, PeepReplace, およびその補助クラス (PeepChild)](no1ZpVCjoB.html))
hotspot/src/share/vm/adlc/formsopt.hpp             |  同上
hotspot/src/share/vm/adlc/formssel.cpp             |  Form のサブクラスの定義 ([InstructForm, EncodeForm, EncClass, MachNodeForm, Opcode, InsEncode, Effect, ExpandRule, RewriteRule, OpClassForm, OperandForm, Constraint, Predicate, Interface, RegInterface, ConstInterface, MemInterface, CondInterface, ConstructRule, AttributeForm, Component, MatchNode, MatchRule, Attribute, FormatRule](nojR9ylYzG.html))
hotspot/src/share/vm/adlc/formssel.hpp             |  同上
hotspot/src/share/vm/adlc/main.cpp                 |  ADLC の main 関数およびその補助関数の定義
hotspot/src/share/vm/adlc/output_c.cpp             |  ArchDesc クラスの cpp ファイル出力用メソッド(およびその補助関数/補助クラス)の定義  ([DefineEmitState, OutputReduceOp, OutputLeftOp, OutputRightOp, OutputRuleName, OutputSwallowed, OutputInstChainRule](noWGD6OEpW.html))  (※1)
hotspot/src/share/vm/adlc/output_h.cpp             |  ArchDesc クラスの hpp ファイル出力用メソッド(およびその補助関数/補助クラス)の定義  ([OutputMachOperands, OutputMachOpcodes](noYbf_vuef.html))  (※2)

### 備考(Notes)
* ※1:
  出力用メソッドとは ArchDesc::defineClasses() 等のこと. 実際に出力する際に main() 内で呼び出されている.
* ※2:
  出力用メソッドとは ArchDesc::declareClasses() 等のこと. 実際に出力する際に main() 内で呼び出されている.






