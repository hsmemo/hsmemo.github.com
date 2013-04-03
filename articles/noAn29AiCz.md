---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： asm/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： asm/

--- 

File Name                                          | Description
-------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/asm/assembler.cpp             | AbstractAssembler クラス関連のクラスの定義 ([Label, RegisterOrConstant, AbstractAssembler, AbstractAssembler::InstructionMark](norF2cpzVJ.html))
hotspot/src/share/vm/asm/assembler.hpp             | 同上
hotspot/src/share/vm/asm/assembler.inline.hpp      | 同上
hotspot/src/share/vm/asm/codeBuffer.cpp            | CodeBuffer クラス関連のクラスの定義 ([CodeOffsets, CodeSection, CodeComments, CodeBuffer, 及びそれらの補助クラス(CodeComment)](nosbVAXUdb.html))
hotspot/src/share/vm/asm/codeBuffer.hpp            | 同上
hotspot/src/share/vm/asm/register.cpp              | AbstractRegisterImpl クラスの定義、およびレジスタ操作時のassert用マクロ定義(assert_different_registers等) ([AbstractRegisterImpl](noz8Ujd3dK.html)) (※1) 
hotspot/src/share/vm/asm/register.hpp              | 同上

### 備考(Notes)
* ※1:
  cpp の方は #include 以外は何も書いていない






