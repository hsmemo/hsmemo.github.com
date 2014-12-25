---
layout: default
title: AbstractRegisterImpl クラス(AbstractRegister) 
---
[Top](../index.html)

#### AbstractRegisterImpl クラス(AbstractRegister) 



---
## <a name="nouzk7peXS" id="nouzk7peXS">AbstractRegisterImpl</a>

### 概要(Summary)
Assembler クラス用の補助クラス.

オペランドとしての「レジスタ」を表すクラスの基底クラス (See: [here](no7882z5r.html) for details).

なお, 具体的な RegisterImpl クラス (AbstractRegisterImpl のサブクラス) は cpu/ 下で定義されている
(See: hotspot/src/cpu/${cpu}/vm/register_${cpu}.hpp).


```cpp
    ((cite: hotspot/src/share/vm/asm/register.hpp))
    // The super class for platform specific registers. Instead of using value objects,
    // registers are implemented as pointers. Subclassing is used so all registers can
    // use the debugging suport below. No virtual functions are used for efficiency.
    // They are canonicalized; i.e., registers are equal if their pointers are equal,
    // and vice versa. A concrete implementation may just map the register onto 'this'.
    
    class AbstractRegisterImpl {
```

なお AbstractRegister という型も使われるが, これは AbstractRegisterImpl* のこと.


```cpp
    ((cite: hotspot/src/share/vm/asm/register.hpp))
    typedef AbstractRegisterImpl* AbstractRegister;
```




### 詳細(Details)
See: [here](../doxygen/classAbstractRegisterImpl.html) for details

---
