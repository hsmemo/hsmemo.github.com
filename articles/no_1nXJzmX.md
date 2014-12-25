---
layout: default
title: markOopDesc クラス 
---
[Top](../index.html)

#### markOopDesc クラス 



---
## <a name="noDza8RM6J" id="noDza8RM6J">markOopDesc</a>

### 概要(Summary)
Java ヒープ中のオブジェクト(oop)のロック管理やハッシュ値管理, GC 処理等で使用される 1word のデータ構造
(See: [here](no2114NIs.html) and [here](no6897XsM.html) for details)

なおコメントによると, 
実際には oopDesc ではなく単なる word だが歴史的な事情により oop hierarchy に属している, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/oops/markOop.hpp))
    // The markOop describes the header of an object.
    //
    // Note that the mark is not a real oop but just a word.
    // It is placed in the oop hierarchy for historical reasons.
```


```cpp
    ((cite: hotspot/src/share/vm/oops/markOop.hpp))
    class markOopDesc: public oopDesc {
```

### 使われ方(Usage)
全ての oopDesc の先頭 1 word 分の領域に埋め込まれている (See: oopDesc).


```cpp
    ((cite: hotspot/src/share/vm/oops/oop.hpp))
    class oopDesc {
    ...
      volatile markOop  _mark;
```

### 内部構造(Internal structure)
markOopDesc の 1 word は以下のように使用される. (#TODO CMS 時の説明) (#TODO UseCompressedOop 時の説明)

* 下位 2bits or 3bits はそのオブジェクトのロック状態を表す
  (biased locking 使用時は 3bits, 非使用時は 2bits).

  残りの部分(上位 62or61/29or30 bits) はロック状態に応じて変わる
  (ロック最適化用の情報, java.lang.Object.hashCode() が使用するハッシュ値, GC が利用する age 情報, 等).
  
* 下位3bitsが 001 なら neutral 状態.
  
  この場合, 上位bits には hashcode や age 情報が埋め込まれている.

  ```
  unused:0/25 hash:25/31 age:4 biased_lock:1 lock:2 = 32/64 bits
  ```

  (<= 32bits なら 0+25+4+1+2, 64bits なら 25+31+4+1+2 と言いたいのだと思うが,
   64bits の方は合計が 63bits になっているのは何故?? #TODO)

* 下位2bitsが 00 なら stack-locked 状態.

  この場合, 上位bits には BasicObjectLock オブジェクトのポインタが格納されている
  (ポインタ値の下位2bitsは 0 なので, 上位30/62bits しか使えなくても問題はない).

  元の hashcode や age 情報はこの BasicObjectLock の内部に退避されている
  (See: BasicObjectLock).

* 下位2bitsが 10 なら inflated (locked/unlocked) 状態.

  この場合, 上位bits には ObjectMonitor オブジェクトのポインタが格納されている
  (ポインタ値の下位2bitsは 0 なので, 上位30/62bits しか使えなくても問題はない).

  元の hashcode や age 情報はこの ObjectMonitor の内部に退避されている.
  (See: ObjectMonitor).

  なお inflated 状態では locked/unlocked の情報は 
  mark フィールドではなく ObjectMonitor 内部に記録されている.

* 下位2bitsが 11 の状態は Garbage Collector が使用する状態.
  
  (marking 処理によってマークされたオブジェクトが 11 になる. これにより既にマーク済みかどうかを区別している)

* 下位3bitsが 101 なら (anonymously) biased 状態.

  この場合, 上位bits には bias 対象の JavaThread オブジェクトのポインタ及び age 情報が格納されている.
  JavaThread のポインタ部分に 0 が入っていた場合は anonymously biased 状態.

  ```
  JavaThread*:23/55 epoch:2 age:4 biased_lock:1 lock:2
  ```

  (なお, biased locking を使用する場合は
   JavaThread オブジェクトのアドレスは (このスペースに収まるよう) 2^9 bits (=1024bits) 境界にアラインメントされる
   (See: Thread::operator new()))

  (なお, biased 状態ではスペースに余裕がないので hashcode を入れておくことができない.
  そのため hashcode が設定された場合は強制的に biased が解除される)


```cpp
    ((cite: hotspot/src/share/vm/oops/markOop.hpp))
    // Bit-format of an object header (most significant first, big endian layout below):
    //
    //  32 bits:
    //  --------
    //             hash:25 ------------>| age:4    biased_lock:1 lock:2 (normal object)
    //             JavaThread*:23 epoch:2 age:4    biased_lock:1 lock:2 (biased object)
    //             size:32 ------------------------------------------>| (CMS free block)
    //             PromotedObject*:29 ---------->| promo_bits:3 ----->| (CMS promoted object)
    //
    //  64 bits:
    //  --------
    //  unused:25 hash:31 -->| unused:1   age:4    biased_lock:1 lock:2 (normal object)
    //  JavaThread*:54 epoch:2 unused:1   age:4    biased_lock:1 lock:2 (biased object)
    //  PromotedObject*:61 --------------------->| promo_bits:3 ----->| (CMS promoted object)
    //  size:64 ----------------------------------------------------->| (CMS free block)
    //
    //  unused:25 hash:31 -->| cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && normal object)
    //  JavaThread*:54 epoch:2 cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && biased object)
    //  narrowOop:32 unused:24 cms_free:1 unused:4 promo_bits:3 ----->| (COOPs && CMS promoted object)
    //  unused:21 size:35 -->| cms_free:1 unused:7 ------------------>| (COOPs && CMS free block)
    //
    //  - hash contains the identity hash value: largest value is
    //    31 bits, see os::random().  Also, 64-bit vm's require
    //    a hash value no bigger than 32 bits because they will not
    //    properly generate a mask larger than that: see library_call.cpp
    //    and c1_CodePatterns_sparc.cpp.
    //
    //  - the biased lock pattern is used to bias a lock toward a given
    //    thread. When this pattern is set in the low three bits, the lock
    //    is either biased toward a given thread or "anonymously" biased,
    //    indicating that it is possible for it to be biased. When the
    //    lock is biased toward a given thread, locking and unlocking can
    //    be performed by that thread without using atomic operations.
    //    When a lock's bias is revoked, it reverts back to the normal
    //    locking scheme described below.
    //
    //    Note that we are overloading the meaning of the "unlocked" state
    //    of the header. Because we steal a bit from the age we can
    //    guarantee that the bias pattern will never be seen for a truly
    //    unlocked object.
    //
    //    Note also that the biased state contains the age bits normally
    //    contained in the object header. Large increases in scavenge
    //    times were seen when these bits were absent and an arbitrary age
    //    assigned to all biased objects, because they tended to consume a
    //    significant fraction of the eden semispaces and were not
    //    promoted promptly, causing an increase in the amount of copying
    //    performed. The runtime system aligns all JavaThread* pointers to
    //    a very large value (currently 128 bytes (32bVM) or 256 bytes (64bVM))
    //    to make room for the age bits & the epoch bits (used in support of
    //    biased locking), and for the CMS "freeness" bit in the 64bVM (+COOPs).
    //
    //    [JavaThread* | epoch | age | 1 | 01]       lock is biased toward given thread
    //    [0           | epoch | age | 1 | 01]       lock is anonymously biased
    //
    //  - the two lock bits are used to describe three states: locked/unlocked and monitor.
    //
    //    [ptr             | 00]  locked             ptr points to real header on stack
    //    [header      | 0 | 01]  unlocked           regular object header
    //    [ptr             | 10]  monitor            inflated lock (header is wapped out)
    //    [ptr             | 11]  marked             used by markSweep to mark an object
    //                                               not valid at any other time
    //
    //    We assume that stack/thread pointers have the lowest two bits cleared.
```

### 備考(Notes)
なお, 実際の使用箇所では markOop という別名(もしくはラッパークラス)で使われることが多い (See: markOop).




### 詳細(Details)
See: [here](../doxygen/classmarkOopDesc.html) for details

---
