---
layout: default
title: RegisterImpl(Register) クラス関連のクラス (RegisterImpl, FloatRegisterImpl, XMMRegisterImpl, ConcreteRegisterImpl)
---
[Top](../index.html)

#### RegisterImpl(Register) クラス関連のクラス (RegisterImpl, FloatRegisterImpl, XMMRegisterImpl, ConcreteRegisterImpl)

これらは, x86 におけるレジスタを表すクラス.


### クラス一覧(class list)

  * [RegisterImpl](#noaIHf7r_T)
  * [FloatRegisterImpl](#noF_9X9a2V)
  * [XMMRegisterImpl](#noc4dmKoX_)
  * [ConcreteRegisterImpl](#noASZdPAF2)


---
## <a name="noaIHf7r_T" id="noaIHf7r_T">RegisterImpl</a>

### 概要(Summary)
AbstractRegisterImpl クラスの具象サブクラスの1つ.

このクラスは x86 の汎用レジスタを表す.
1つの RegisterImpl オブジェクトが 1本の汎用レジスタに対応する.


```
    ((cite: hotspot/src/cpu/x86/vm/register_x86.hpp))
    class RegisterImpl: public AbstractRegisterImpl {
```

なお Register という型も使われるが, これは RegisterImpl* の別名.


```
    ((cite: hotspot/src/cpu/x86/vm/register_x86.hpp))
    typedef RegisterImpl* Register;
```

### 内部構造(Internal structure)
内部には 1つの int 値だけを持つ

なお int 型のフィールドを持つわけではなく, 
int 値を RegisterImpl* にキャストして使用する (See: as_Register()).

(つまり, RegisterImpl クラス内ではこの int 値に "this" でアクセスできる)

(あるいは, int 値にアクセスするための encoding() というアクセサメソッドも用意されている)

#### 参考(for your information): as_Register()
See: [here](no2935IHC.html) for details
### 備考(Notes)
なお, レジスタを表すという意味では VMReg クラスとも少し似ている.

ただし VMReg では 1 オブジェクトが 32bit (固定長) に対応するのに対し, 
RegisterImpl では 1 オブジェクトはレジスタ1つ (環境によって可変) に対応する.

このため VMReg の番号との対応は, 32bit なら恒等写像, 64bit なら2倍した値.




### 詳細(Details)
See: [here](../doxygen/classRegisterImpl.html) for details

---
## <a name="noF_9X9a2V" id="noF_9X9a2V">FloatRegisterImpl</a>

### 概要(Summary)
AbstractRegisterImpl クラスの具象サブクラスの1つ.

このクラスは x86 の浮動小数レジスタ(x87 レジスタ)を表す.
1つの FloatRegisterImpl オブジェクトが 1本の浮動小数レジスタに対応する.


```
    ((cite: hotspot/src/cpu/x86/vm/register_x86.hpp))
    // The implementation of floating point registers for the ia32 architecture
    class FloatRegisterImpl: public AbstractRegisterImpl {
```

なお FloatRegister という型も使われるが, これは FloatRegisterImpl* の別名.


```
    ((cite: hotspot/src/cpu/x86/vm/register_x86.hpp))
    typedef FloatRegisterImpl* FloatRegister;
```

### 内部構造(Internal structure)
内部には 1つの int 値だけを持つ.

なお int 型のフィールドを持つわけではなく, 
int 値を FloatRegisterImpl* にキャストして使用する (See: as_FloatRegister().

(つまり, FloatRegisterImpl クラス内ではこの int 値に "this" でアクセスできる)

(あるいは, int 値にアクセスするための encoding() というアクセサメソッドも用意されている)

#### 参考(for your information): as_FloatRegister()
See: [here](no2935VRI.html) for details
### 備考(Notes)
なお, レジスタを表すという意味では VMReg クラスとも少し似ている.

ただし VMReg では 1 オブジェクトが 32bit (固定長) に対応するのに対し, 
FloatRegisterImpl では 1 オブジェクトは 64bit (固定長) に対応する.

このため VMReg の番号との対応は, 2倍した値+ConcreteRegisterImpl::max_gpr.




### 詳細(Details)
See: [here](../doxygen/classFloatRegisterImpl.html) for details

---
## <a name="noc4dmKoX_" id="noc4dmKoX_">XMMRegisterImpl</a>

### 概要(Summary)
AbstractRegisterImpl クラスの具象サブクラスの1つ.

このクラスは x86 の XMM レジスタを表す.
1つの XMMRegisterImpl オブジェクトが 1本の XMM レジスタに対応する.


```
    ((cite: hotspot/src/cpu/x86/vm/register_x86.hpp))
    // The implementation of XMM registers for the IA32 architecture
    class XMMRegisterImpl: public AbstractRegisterImpl {
```

なお XMMRegister という型も使われるが, これは XMMRegisterImpl* の別名.


```
    ((cite: hotspot/src/cpu/x86/vm/register_x86.hpp))
    typedef XMMRegisterImpl* XMMRegister;
```

### 内部構造(Internal structure)
内部には 1つの int 値だけを持つ.

なお int 型のフィールドを持つわけではなく, 
int 値を XMMRegisterImpl* にキャストして使用する (See: as_XMMRegister()).

(つまり, XMMRegisterImpl クラス内ではこの int 値に "this" でアクセスできる)

(あるいは, int 値にアクセスするための encoding() というアクセサメソッドも用意されている)

#### 参考(for your information): as_XMMRegister()
See: [here](no29358va.html) for details
### 備考(Notes)
なお, レジスタを表すという意味では VMReg クラスとも少し似ている.

ただし VMReg では 1 オブジェクトが 32bit (固定長) に対応するのに対し, 
XMMRegisterImpl では 1 オブジェクトは 64bit (固定長) に対応する.

このため VMReg の番号との対応は, 2倍した値+ConcreteRegisterImpl::max_fpr.




### 詳細(Details)
See: [here](../doxygen/classXMMRegisterImpl.html) for details

---
## <a name="noASZdPAF2" id="noASZdPAF2">ConcreteRegisterImpl</a>

### 概要(Summary)
レジスタの本数に関する定数値を納めた名前空間
(このクラスは AllStatic ではないが, static な定義しか持たない).

汎用レジスタ, 浮動小数レジスタ, XMM レジスタの個数を記録している.


```
    ((cite: hotspot/src/cpu/x86/vm/register_x86.hpp))
    // Need to know the total number of registers of all sorts for SharedInfo.
    // Define a class that exports it.
    class ConcreteRegisterImpl : public AbstractRegisterImpl {
```

### 内部構造(Internal structure)
内部には以下の定数定義およびフィールド定義(のみ)を含む.


```
    ((cite: hotspot/src/cpu/x86/vm/register_x86.hpp))
      enum {
      // A big enough number for C2: all the registers plus flags
      // This number must be large enough to cover REG_COUNT (defined by c2) registers.
      // There is no requirement that any ordering here matches any ordering c2 gives
      // it's optoregs.
    
        number_of_registers =      RegisterImpl::number_of_registers +
    #ifdef AMD64
                                   RegisterImpl::number_of_registers +  // "H" half of a 64bit register
    #endif // AMD64
                               2 * FloatRegisterImpl::number_of_registers +
                               2 * XMMRegisterImpl::number_of_registers +
                               1 // eflags
      };
```


```
    ((cite: hotspot/src/cpu/x86/vm/register_x86.hpp))
      static const int max_gpr;
      static const int max_fpr;
      static const int max_xmm;
```




### 詳細(Details)
See: [here](../doxygen/classConcreteRegisterImpl.html) for details

---
