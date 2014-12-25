---
layout: default
title: PSVirtualSpace クラス関連のクラス (PSVirtualSpace, PSVirtualSpaceVerifier, PSVirtualSpaceHighToLow)
---
[Top](../index.html)

#### PSVirtualSpace クラス関連のクラス (PSVirtualSpace, PSVirtualSpaceVerifier, PSVirtualSpaceHighToLow)



### クラス一覧(class list)

  * [PSVirtualSpace](#noMjmAkmzc)
  * [PSVirtualSpaceHighToLow](#no9D9ZRHyA)
  * [PSVirtualSpaceVerifier](#noiYfvYz3q)


---
## <a name="noMjmAkmzc" id="noMjmAkmzc">PSVirtualSpace</a>

### 概要(Summary)
ParallelScavenge 用の VirtualSpace クラス (See: VirtualSpace).

なお, VirtualSpace クラスと異なり, このクラスは _ValueObj クラスではなく CHeapObj クラスになっている 
(というわけでこのクラスは VirtualSpace クラスのサブクラスでもない).

(わざわざ VirtualSpace クラスとは別に PSVirtualSpace クラスを作った理由は, 
継承によって PSVirtualSpace オブジェクトと PSVirtualSpaceHighToLow オブジェクトを統一的に扱いたかったからだと思われる.
PSVirtualSpace オブジェクトと(そのサブクラスである) PSVirtualSpaceHighToLow オブジェクトは, どちらも PSVirtualSpace* 型のフィールドに入る.
VirtualSpace クラスやそのサブクラスでは _ValueObj クラスになってしまうので(= ポインタ経由で扱えないので)こうしたテクニックが使えない.)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psVirtualspace.hpp))
    // VirtualSpace for the parallel scavenge collector.
    //
    // VirtualSpace is data structure for committing a previously reserved address
    // range in smaller chunks.
    
    class PSVirtualSpace : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classPSVirtualSpace.html) for details

---
## <a name="no9D9ZRHyA" id="no9D9ZRHyA">PSVirtualSpaceHighToLow</a>

### 概要(Summary)
特殊な PSVirtualSpace クラス.
(通常の PSVirtualSpace とは逆に) expand_by() 時にアドレス空間上の下方に向かって伸びる.

(なぜ下方に伸びないといけないかは AdjoiningVirtualSpaces を参照)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psVirtualspace.hpp))
    // A virtual space that grows from high addresses to low addresses.
    class PSVirtualSpaceHighToLow : public PSVirtualSpace {
```

### 使われ方(Usage)
ASPSYoungGen 内で使用されている.




### 詳細(Details)
See: [here](../doxygen/classPSVirtualSpaceHighToLow.html) for details

---
## <a name="noiYfvYz3q" id="noiYfvYz3q">PSVirtualSpaceVerifier</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psVirtualspace.hpp))
    #ifndef PRODUCT
    ...
      // Helper class to verify a space when entering/leaving a block.
      class PSVirtualSpaceVerifier: public StackObj {
```

コンストラクタ引数で指定された PSVirtualSpace オブジェクトの検証を行う
(コンストラクタとデストラクタで PSVirtualSpace::verify() を呼び出している).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psVirtualspace.hpp))
        PSVirtualSpaceVerifier(PSVirtualSpace* space): _space(space) {
          _space->verify();
        }
        ~PSVirtualSpaceVerifier() { _space->verify(); }
```

### 使われ方(Usage)
PSVirtualSpace クラス (や PSVirtualSpaceHighToLow クラス) の以下のメソッド内で使用されている.

* PSVirtualSpace::release()
* PSVirtualSpace::expand_by()
* PSVirtualSpace::shrink_by()
* PSVirtualSpace::expand_into()
* PSVirtualSpaceHighToLow::expand_by()
* PSVirtualSpaceHighToLow::shrink_by()
* PSVirtualSpaceHighToLow::expand_into()

(ただし, DEBUG_ONLY で囲われているので, 使われるのはデバッグ時のみ).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psVirtualspace.cpp))
    void PSVirtualSpace::release() {
      DEBUG_ONLY(PSVirtualSpaceVerifier this_verifier(this));
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psVirtualspace.cpp))
    bool PSVirtualSpace::expand_by(size_t bytes) {
    ...
      DEBUG_ONLY(PSVirtualSpaceVerifier this_verifier(this));
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psVirtualspace.cpp))
    bool PSVirtualSpace::shrink_by(size_t bytes) {
    ...
      DEBUG_ONLY(PSVirtualSpaceVerifier this_verifier(this));
```




### 詳細(Details)
See: [here](../doxygen/classPSVirtualSpaceVerifier.html) for details

---
