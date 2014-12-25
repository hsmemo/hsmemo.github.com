---
layout: default
title: Method に関する処理 ： ランタイムを呼び出す処理 ： Template Interpreter の場合  
---
[Up](nouRrdNbjG.html) [Top](../index.html)

#### Method に関する処理 ： ランタイムを呼び出す処理 ： Template Interpreter の場合  

--- 
## 概要(Summary)
Template Interpreter がランタイム(InterpreterRuntime, SharedRuntime)を呼び出す場合,
以下の関数が生成するコードが使用される.

  * MacroAssembler::call_VM()
    
    非リーフ関数用  (JRT_ENTRY で宣言されているもの等.  See: [here](no3059XPe.html) for details)
   
  * MacroAssembler::call_VM_leaf()
   
    リーフ関数用    (JRT_LEAF で宣言されているもの等.   See: [here](no3059XPe.html) for details)


```cpp
    ((cite: hotspot/src/cpu/x86/vm/assembler_x86.hpp))
      // Support for VM calls
      //
      // It is imperative that all calls into the VM are handled via the call_VM macros.
      // They make sure that the stack linkage is setup correctly. call_VM's correspond
      // to ENTRY/ENTRY_X entry points while call_VM_leaf's correspond to LEAF entry points.
```


### MacroAssembler::call_VM()
MacroAssembler::call_VM() には, 呼び出すランタイム関数の引数の個数に応じて 4通りが存在する
  (引数の数が 1個, 2個, 3個, 及びそれらから呼び出される可変個版).
また, last_java_sp を示すレジスタを渡すものと渡さないものがある.
このため, 全体では 4*2 = 8通りの call_VM() 関数が用意されている.

ただし, どの場合も最終的には MacroAssembler::call_VM_base() 内で実際の呼び出し処理が行われる.


なお, 現状の sparc 版では, last_java_sp を渡すバージョンは使われていないとのこと.


```cpp
    ((cite: hotspot/src/cpu/sparc/vm/assembler_sparc.hpp))
      // these overloadings are not presently used on SPARC:
```

また, last_java_sp を渡すバージョンは, 
既に save が実行されていて last Java frame が一つ前のフレームになっている場合に便利, とのこと
(この場合, last_javva_sp は SP ではなく FP として渡される必要がある).


```cpp
    ((cite: hotspot/src/cpu/sparc/vm/assembler_sparc.cpp))
    // Note: The following call_VM overloadings are useful when a "save"
    // has already been performed by a stub, and the last Java frame is
    // the previous one.  In that case, last_java_sp must be passed as FP
    // instead of SP.
```

### MacroAssembler::call_VM_leaf()
MacroAssembler::call_VM_leaf() も MacroAssembler::call_VM() と同様で,
呼び出すランタイム関数の引数の個数に応じて 4通りが存在する
(それぞれ, 引数の数が 1個, 2個, 3個, 及びそれらから呼び出される可変個版).

どの場合も, 最終的には MacroAssembler::call_VM_leaf_base() 内で実際の呼び出し処理が行われる


## 備考(Notes)
VMルーチンはC++コードであるため, 処理としては native メソッドの呼び出しに若干似ている.

## 処理の流れ (概要)(Execution Flows : Summary)
### sparc の場合
#### MacroAssembler::call_VM() (last_java_sp 指定無し) の場合
```
MacroAssembler::call_VM() (引数 1~3 個版)  が生成するコード
-> MacroAssembler::call_VM() (引数可変個版)  が生成するコード
   -> InterpreterMacroAssembler::call_VM_base()  が生成するコード
      -> MacroAssembler::call_VM_base()  が生成するコード
         -> JavaFrameAnchor に, 呼び出し元の Java メソッドの PC と SP を退避
         -> 指定されたエントリポイントを呼び出す
         -> 呼び出し元の Java メソッドの PC と SP を復帰
         -> MacroAssembler::check_and_forward_exception() が生成するコード
            (例外や PopFrame, ForceEarlyReturn のチェックを行う. 必要に応じてそれぞれの処理ルーチンにジャンプ)
            -> InterpreterMacroAssembler::check_and_handle_popframe()  が生成するコード
               -> (See: [here](no2935cDo.html) for details)
            -> InterpreterMacroAssembler::check_and_handle_earlyret()  が生成するコード
               -> (See: [here](no3059azN.html) for details)
            -> 例外チェックを行い, 発生していれば例外処理ルーチンへジャンプ (StubRoutines::forward_exception_entry()).
               -> (See: [here](no293560A.html) for details)
```

#### MacroAssembler::call_VM() (last_java_sp 指定有り) の場合
```
MacroAssembler::call_VM() (引数 1~3 個版, last_java_sp 指定有り)  が生成するコード
-> MacroAssembler::call_VM() (引数可変個版, last_java_sp 指定有り)  が生成するコード
   -> InterpreterMacroAssembler::call_VM_base()  が生成するコード
      -> (同上)
```

#### MacroAssembler::call_VM_leaf() の場合
```
MacroAssembler::call_VM_leaf() (引数 1~3 個版)  が生成するコード
-> MacroAssembler::call_VM_leaf() (引数可変個版)  が生成するコード
   -> InterpreterMacroAssembler::call_VM_leaf_base()  が生成するコード
      -> MacroAssembler::call_VM_leaf_base()  が生成するコード
```

### x86_64 の場合
#### MacroAssembler::call_VM() (last_java_sp 指定無し) の場合
```
MacroAssembler::call_VM() (引数 1~3 個, 及び可変個版)  が生成するコード
-> MacroAssembler::call_VM_helper()  が生成するコード
   -> InterpreterMacroAssembler::call_VM_base()  が生成するコード
      -> MacroAssembler::call_VM_base()  が生成するコード
         -> MacroAssembler::call_VM_leaf_base()  が生成するコード
            -> 指定されたエントリポイントを呼び出す
         -> InterpreterMacroAssembler::check_and_handle_popframe()  が生成するコード
            -> (See: [here](no2935cDo.html) for details)
         -> InterpreterMacroAssembler::check_and_handle_earlyret()  が生成するコード
            -> (See: [here](no3059azN.html) for details)
         -> 例外チェックを行い, 発生していれば例外処理ルーチンへジャンプ (StubRoutines::forward_exception_entry()).
            -> (See: [here](no293560A.html) for details)
```

#### MacroAssembler::call_VM() (last_java_sp 指定有り) の場合
```
MacroAssembler::call_VM() (引数 1~3 個版, last_java_sp 指定有り)  が生成するコード
-> MacroAssembler::call_VM() (引数可変個版, last_java_sp 指定有り)  が生成するコード
   -> InterpreterMacroAssembler::call_VM_base()  が生成するコード
      -> (同上)
```

#### MacroAssembler::call_VM_leaf() の場合
```
MacroAssembler::call_VM_leaf() (引数 1~3 個版)  が生成するコード
-> MacroAssembler::call_VM_leaf() (引数可変個版)  が生成するコード
   -> InterpreterMacroAssembler::call_VM_leaf_base()  が生成するコード
      -> MacroAssembler::call_VM_leaf_base()  が生成するコード
         -> (同上)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### MacroAssembler::call_VM() (引数可変個版) (sparc の場合)
See: [here](no3059VNQ.html) for details
### MacroAssembler::call_VM() (引数 1 個版) (sparc の場合)
See: [here](no3059iXW.html) for details
### MacroAssembler::call_VM() (引数 2 個版) (sparc の場合)
See: [here](no3059vhc.html) for details
### MacroAssembler::call_VM() (引数 3 個版) (sparc の場合)
See: [here](no30598ri.html) for details
### MacroAssembler::call_VM() (引数可変個版, last_java_sp 指定有り) (sparc の場合)
See: [here](no3059WAv.html) for details
### MacroAssembler::call_VM() (引数 1 個版, last_java_sp 指定有り) (sparc の場合)
See: [here](no3059jK1.html) for details
### MacroAssembler::call_VM() (引数 2 個版, last_java_sp 指定有り) (sparc の場合)
See: [here](no3059VUE.html) for details
### MacroAssembler::call_VM() (引数 3 個版, last_java_sp 指定有り) (sparc の場合)
See: [here](no3059ieK.html) for details
### InterpreterMacroAssembler::call_VM_base() (sparc の場合)
See: [here](no3059tmC.html) for details
### MacroAssembler::call_VM_base() (sparc の場合)
See: [here](no30596pU.html) for details
### MacroAssembler::set_last_Java_frame() (sparc の場合)
See: [here](no4230FRA.html) for details
### MacroAssembler::reset_last_Java_frame() (sparc の場合)
See: [here](no4230SbG.html) for details
### MacroAssembler::check_and_forward_exception() (sparc の場合)
See: [here](no3059H0a.html) for details
### MacroAssembler::get_vm_result() (sparc の場合)
See: [here](no30597cz.html) for details

### MacroAssembler::call_VM_leaf() (引数可変個版) (sparc の場合)
See: [here](no30598yW.html) for details
### MacroAssembler::call_VM_leaf() (引数 1 個版) (sparc の場合)
See: [here](no3059J9c.html) for details
### MacroAssembler::call_VM_leaf() (引数 2 個版) (sparc の場合)
See: [here](no3059WHj.html) for details
### MacroAssembler::call_VM_leaf() (引数 3 個版) (sparc の場合)
See: [here](no3059jRp.html) for details
### InterpreterMacroAssembler::call_VM_leaf_base() (sparc の場合)
See: [here](no305996R.html) for details
### MacroAssembler::call_VM_leaf_base() (sparc の場合)
See: [here](no3059voQ.html) for details

### MacroAssembler::call_VM() (引数可変個版) (x86 の場合)
See: [here](no3059vvE.html) for details
### MacroAssembler::call_VM() (引数 1 個版) (x86 の場合)
See: [here](no3059K3v.html) for details
### MacroAssembler::pass_arg1() (x86_32 の場合)
See: [here](no3059WVL.html) for details
### MacroAssembler::pass_arg1() (x86_64 の場合)
See: [here](no30599zd.html) for details
### MacroAssembler::call_VM() (引数 2 個版) (x86 の場合)
See: [here](no3059XB2.html) for details
### MacroAssembler::pass_arg2() (x86_32 の場合)
See: [here](no3059jfR.html) for details
### MacroAssembler::pass_arg2() (x86_64 の場合)
See: [here](no3059K-j.html) for details
### MacroAssembler::call_VM() (引数 3 個版) (x86 の場合)
See: [here](no3059JLF.html) for details
### MacroAssembler::pass_arg3() (x86_32 の場合)
See: [here](no3059wpX.html) for details
### MacroAssembler::pass_arg3() (x86_64 の場合)
See: [here](no3059XIq.html) for details
### MacroAssembler::call_VM_helper() (x86 の場合)
See: [here](no3059wij.html) for details
### MacroAssembler::call_VM() (引数可変個版, last_java_sp 指定有り) (x86 の場合)
See: [here](no305985K.html) for details
### MacroAssembler::call_VM() (引数 1 個版, last_java_sp 指定有り) (x86 の場合)
See: [here](no3059JER.html) for details
### MacroAssembler::call_VM() (引数 2 個版, last_java_sp 指定有り) (x86 の場合)
See: [here](no3059WOX.html) for details
### MacroAssembler::call_VM() (引数 3 個版, last_java_sp 指定有り) (x86 の場合)
See: [here](no3059jYd.html) for details
### InterpreterMacroAssembler::call_VM_base() (x86_64 の場合)
See: [here](no3059UFV.html) for details
### MacroAssembler::call_VM_base() (x86 の場合)
See: [here](no30596wI.html) for details
### MacroAssembler::set_last_Java_frame() (x86 の場合)
See: [here](no4230flM.html) for details
### MacroAssembler::reset_last_Java_frame() (x86 の場合)
See: [here](no4230svS.html) for details

### MacroAssembler::call_VM_leaf() (引数可変個版) (x86 の場合)
See: [here](no3059kSw.html) for details
### MacroAssembler::call_VM_leaf() (引数 1 個版) (x86 の場合)
See: [here](no3059xc2.html) for details
### MacroAssembler::call_VM_leaf() (引数 2 個版) (x86 の場合)
See: [here](no3059jmF.html) for details
### MacroAssembler::call_VM_leaf() (引数 3 個版) (x86 の場合)
See: [here](no3059wwL.html) for details
### InterpreterMacroAssembler::call_VM_leaf_base() (x86_64 の場合)
See: [here](no3059KFY.html) for details
### MacroAssembler::call_VM_leaf_base() (x86 の場合)
See: [here](no3059H7O.html) for details
#### 備考(Notes)
なお同じファイル内に #ifndef _LP64 版とそうでない版がある. これは _LP64 版.

### InterpreterMacroAssembler::restore_bcp() (x86_64 の場合)
See: [here](no30597jn.html) for details






