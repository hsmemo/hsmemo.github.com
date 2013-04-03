---
layout: default
title: InterpreterMacroAssembler クラス 
---
[Top](../index.html)

#### InterpreterMacroAssembler クラス 



---
## <a name="nod5ydSODu" id="nod5ydSODu">InterpreterMacroAssembler</a>

### 概要(Summary)
Interpreter 用に拡張された MacroAssembler クラス.

(例えば, Java のコードを呼び出す InterpreterMacroAssembler::call_VM_base() や
次のバイトコードをディスパッチする InterpreterMacroAssembler::dispatch_next(), 
スタックフレームを破棄する InterpreterMacroAssembler::remove_activation() 等, 
インタープリタ用の複雑なアセンブリ生成ルーチンが用意されている)

(なお, 32bit か 64bit かによってクラス定義が別になっている)


```
    ((cite: hotspot/src/cpu/x86/vm/interp_masm_x86_32.hpp))
    // This file specializes the assember with interpreter-specific macros
    
    
    class InterpreterMacroAssembler: public MacroAssembler {
```


```
    ((cite: hotspot/src/cpu/x86/vm/interp_masm_x86_64.hpp))
    // This file specializes the assember with interpreter-specific macros
    
    
    class InterpreterMacroAssembler: public MacroAssembler {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

(このクラスは ResourceObj クラスなので永続的に存在するわけではないが, 
 コード生成作業の間は以下のフィールドに格納されている)

* 各 CodeletMark オブジェクトの _masm フィールド

* TemplateInterpreterGenerator オブジェクトの _masm フィールド

  (ただし, TemplateInterpreterGenerator::set_entry_points() 及び
   TemplateInterpreterGenerator::generate_all() 中のコード生成の間のみ.
   また, 格納している InterpreterMacroAssembler オブジェクトは, 
   これらの中で使用する CodeletMark オブジェクトの _masm フィールドに格納されているものと同一)

* CppInterpreterGenerator オブジェクトの _masm フィールド

  (ただし, CppInterpreterGenerator::generate_all() 中のコード生成の間のみ.
   また, 格納している InterpreterMacroAssembler オブジェクトは, 
   この中で使用する CodeletMark オブジェクトの _masm フィールドに格納されているものと同一)

* AbstractInterpreterGenerator オブジェクトの _masm フィールド

  (ただし, AbstractInterpreterGenerator::generate_all() 中のコード生成の間のみ.
   また, 格納している InterpreterMacroAssembler オブジェクトは, 
   この中で使用する CodeletMark オブジェクトの _masm フィールドに格納されているものと同一)

#### 生成箇所(where its instances are created)
CodeletMark::CodeletMark() 内で(のみ)生成されている
(See: CodeletMark).




### 詳細(Details)
See: [here](../doxygen/classInterpreterMacroAssembler.html) for details

---
