---
layout: default
title: Bytecode クラス関連のクラス (Bytecode, LookupswitchPair, Bytecode_lookupswitch, Bytecode_tableswitch, Bytecode_member_ref, Bytecode_invoke, Bytecode_field, Bytecode_checkcast, Bytecode_instanceof, Bytecode_new, Bytecode_multianewarray, Bytecode_anewarray, Bytecode_loadconstant)
---
[Top](../index.html)

#### Bytecode クラス関連のクラス (Bytecode, LookupswitchPair, Bytecode_lookupswitch, Bytecode_tableswitch, Bytecode_member_ref, Bytecode_invoke, Bytecode_field, Bytecode_checkcast, Bytecode_instanceof, Bytecode_new, Bytecode_multianewarray, Bytecode_anewarray, Bytecode_loadconstant)

これらは, メソッド中の各バイトコード命令の情報にアクセスするためのユーティリティ・クラス.

### 概要(Summary)
これらは, バイトコード情報を用いた作業中に使用されるユーティリティ・クラス.

例えば, bytecode 中から指定したオペランドを取り出すメソッド(Bytecode::get_index_u1(), Bytecode::get_index_u2(), etc)や,
指定の bytecode の命令長を取得するメソッド(Bytecode::instruction_size()) 等が用意されている.

また, Bytecode クラスのサブクラス (Bytecode_lookupswitch, Bytecode_tableswitch, Bytecode_member_ref, etc) では,
それぞれの命令のオペランドに特化した取得メソッドが提供されている.

例: lookupswitch 命令の場合 (デフォルトのオフセット, ペア数, n 番目のペア, 等の情報を取得するメソッドが用意されている)


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
      // Attributes
      int  default_offset() const                    { return get_Java_u4_at(aligned_offset(1 + 0*jintSize)); }
      int  number_of_pairs() const                   { return get_Java_u4_at(aligned_offset(1 + 1*jintSize)); }
      LookupswitchPair pair_at(int i) const          {
        assert(0 <= i && i < number_of_pairs(), "pair index out of bounds");
        return LookupswitchPair(aligned_addr_at(1 + (1 + i)*2*jintSize));
      }
```


### クラス一覧(class list)

  * [Bytecode](#no01tOIfqI)
  * [Bytecode_lookupswitch](#noBV0PXYnH)
  * [LookupswitchPair](#noWj711ZXd)
  * [Bytecode_tableswitch](#not46rXDv0)
  * [Bytecode_member_ref](#noJKQmy3LV)
  * [Bytecode_invoke](#no4PS50ufG)
  * [Bytecode_field](#noj9AOUNZV)
  * [Bytecode_checkcast](#noFGgMF8a8)
  * [Bytecode_instanceof](#noXFNbob4x)
  * [Bytecode_new](#noCMpacRwj)
  * [Bytecode_multianewarray](#no1_Rtlicz)
  * [Bytecode_anewarray](#nomEIXbSuv)
  * [Bytecode_loadconstant](#noBSmrK3go)


---
## <a name="no01tOIfqI" id="no01tOIfqI">Bytecode</a>

### 概要(Summary)
メソッド中の各バイトコード命令の情報にアクセスするためのユーティリティ・クラス(StackObjクラス)の基底クラス.

なお, このクラスは abstract class ではない
(ただし StackObj クラスというよりは ValueObj クラスに近い使われ方をしている).


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    // The base class for different kinds of bytecode abstractions.
    // Provides the primitive operations to manipulate code relative
    // to the bcp.
    
    class Bytecode: public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
使う際には, 操作対象としたい bytecode 命令の bcp をコンストラクタに指定してインスタンスを生成する模様
(bcp の指定には, methodOop と address を使用).


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
      Bytecode(methodOop method, address bcp): _bcp(bcp), _code(Bytecodes::code_at(method, addr_at(0))) {
        assert(method != NULL, "this form requires a valid methodOop");
      }
```

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciBytecodeStream::bytecode()
* ciBytecodeStream::next_bytecode()
* BaseBytecodeStream::bytecode()
* InterpreterRuntime::bytecode()




### 詳細(Details)
See: [here](../doxygen/classBytecode.html) for details

---
## <a name="noBV0PXYnH" id="noBV0PXYnH">Bytecode_lookupswitch</a>

### 概要(Summary)
lookupswitch 命令用の Bytecode クラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    class Bytecode_lookupswitch: public Bytecode {
```




### 詳細(Details)
See: [here](../doxygen/classBytecode__lookupswitch.html) for details

---
## <a name="noWj711ZXd" id="noWj711ZXd">LookupswitchPair</a>

### 概要(Summary)
lookupswitch 命令中の match-offset pair オペランドを表すクラス.
1つの LookupswitchPair オブジェクトが 1つの match-offset pair (1 組の match-offset からなる 8byte の情報) に対応する.

(match-offset pair の情報を取得するための LookupswitchPair::match() メソッド, 及び LookupswitchPair::offset() メソッドを備える)


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    // Abstractions for lookupswitch bytecode
    class LookupswitchPair VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
Bytecode_lookupswitch::pair_at() で生成される (他に生成箇所はあるか?? #TODO).




### 詳細(Details)
See: [here](../doxygen/classLookupswitchPair.html) for details

---
## <a name="not46rXDv0" id="not46rXDv0">Bytecode_tableswitch</a>

### 概要(Summary)
tableswitch 命令用の Bytecode クラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    class Bytecode_tableswitch: public Bytecode {
```



### 詳細(Details)
See: [here](../doxygen/classBytecode__tableswitch.html) for details

---
## <a name="noJKQmy3LV" id="noJKQmy3LV">Bytecode_member_ref</a>

### 概要(Summary)
メソッド呼び出し用のバイトコード, およびフィールドアクセス用のバイトコードを表す Bytecode クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    // Common code for decoding invokes and field references.
    
    class Bytecode_member_ref: public Bytecode {
```



### 詳細(Details)
See: [here](../doxygen/classBytecode__member__ref.html) for details

---
## <a name="no4PS50ufG" id="no4PS50ufG">Bytecode_invoke</a>

### 概要(Summary)
メソッド呼び出しに関するバイトコード命令 (invoke{virtual|static|interface|special}) 用の Bytecode クラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    // Abstraction for invoke_{virtual, static, interface, special}
    
    class Bytecode_invoke: public Bytecode_member_ref {
```



### 詳細(Details)
See: [here](../doxygen/classBytecode__invoke.html) for details

---
## <a name="noj9AOUNZV" id="noj9AOUNZV">Bytecode_field</a>

### 概要(Summary)
フィールドアクセスに関するバイトコード命令 ((put|get}{field|static}) 用の Bytecode クラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    // Abstraction for all field accesses (put/get field/static)
    class Bytecode_field: public Bytecode_member_ref {
```



### 詳細(Details)
See: [here](../doxygen/classBytecode__field.html) for details

---
## <a name="noFGgMF8a8" id="noFGgMF8a8">Bytecode_checkcast</a>

### 概要(Summary)
checkcast 命令用の Bytecode クラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    // Abstraction for checkcast
    class Bytecode_checkcast: public Bytecode {
```



### 詳細(Details)
See: [here](../doxygen/classBytecode__checkcast.html) for details

---
## <a name="noXFNbob4x" id="noXFNbob4x">Bytecode_instanceof</a>

### 概要(Summary)
instanceof 命令用の Bytecode クラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    // Abstraction for instanceof
    class Bytecode_instanceof: public Bytecode {
```



### 詳細(Details)
See: [here](../doxygen/classBytecode__instanceof.html) for details

---
## <a name="noCMpacRwj" id="noCMpacRwj">Bytecode_new</a>

### 概要(Summary)
new 命令用の Bytecode クラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    class Bytecode_new: public Bytecode {
```



### 詳細(Details)
See: [here](../doxygen/classBytecode__new.html) for details

---
## <a name="no1_Rtlicz" id="no1_Rtlicz">Bytecode_multianewarray</a>

### 概要(Summary)
multianewarray 命令用の Bytecode クラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    class Bytecode_multianewarray: public Bytecode {
```



### 詳細(Details)
See: [here](../doxygen/classBytecode__multianewarray.html) for details

---
## <a name="nomEIXbSuv" id="nomEIXbSuv">Bytecode_anewarray</a>

### 概要(Summary)
anewarray 命令用の Bytecode クラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    class Bytecode_anewarray: public Bytecode {
```



### 詳細(Details)
See: [here](../doxygen/classBytecode__anewarray.html) for details

---
## <a name="noBSmrK3go" id="noBSmrK3go">Bytecode_loadconstant</a>

### 概要(Summary)
ldc, ldc_w 及び ldc2_w 命令用の Bytecode クラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecode.hpp))
    // Abstraction for ldc, ldc_w and ldc2_w
    class Bytecode_loadconstant: public Bytecode {
```




### 詳細(Details)
See: [here](../doxygen/classBytecode__loadconstant.html) for details

---
