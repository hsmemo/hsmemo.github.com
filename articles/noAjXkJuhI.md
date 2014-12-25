---
layout: default
title: AdlcVMDeps クラス 
---
[Top](../index.html)

#### AdlcVMDeps クラス 



---
## <a name="noBZXgAX7g" id="noBZXgAX7g">AdlcVMDeps</a>

### 概要(Summary)
ADLC と HotSpot 内で共通で使われる定数や補助関数を納めた名前空間(AllStatic クラス).


```cpp
    ((cite: hotspot/src/share/vm/opto/adlcVMDeps.hpp))
    // Declare commonly known constant and data structures between the
    // ADLC and the VM
    //
    
    class AdlcVMDeps : public AllStatic {
```

### 内部構造(Internal structure)
内部には以下の定数定義およびメソッド定義(のみ)を含む.


```cpp
    ((cite: hotspot/src/share/vm/opto/adlcVMDeps.hpp))
      // Mirror of TypeFunc types
      enum { Control, I_O, Memory, FramePtr, ReturnAdr, Parms };
    
      enum Cisc_Status { Not_cisc_spillable = -1 };
    
      // Mirror of OptoReg::Name names
      enum Name {
        Physical = 0                // Start of physical regs
      };
    
      // relocInfo
      static const char* oop_reloc_type()  { return "relocInfo::oop_type"; }
      static const char* none_reloc_type() { return "relocInfo::none"; }
```





### 詳細(Details)
See: [here](../doxygen/classAdlcVMDeps.html) for details

---
