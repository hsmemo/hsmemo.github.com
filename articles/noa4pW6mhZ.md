---
layout: default
title: DTrace クラス (DTrace, 及びその補助クラス(VM_DeoptimizeTheWorld))
---
[Top](../index.html)

#### DTrace クラス (DTrace, 及びその補助クラス(VM_DeoptimizeTheWorld))



### クラス一覧(class list)

  * [DTrace](#nopv6RrM2h)
  * [VM_DeoptimizeTheWorld](#nougXBkwDc)


---
## <a name="nopv6RrM2h" id="nopv6RrM2h">DTrace</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する serviceability 機能からのみ使用される).

Solaris 版の AttachListener クラス内で使用されている補助クラス(AllStatic クラス).


```
    ((cite: hotspot/src/share/vm/services/dtraceAttacher.hpp))
    class DTrace : public AllStatic {
```

より具体的に言うと, 以下の DTrace 関連のコマンドラインオプションの値を実行中に変えるためのクラス.

* DTraceAllocProbes
* DTraceMethodProbes
* DTraceMonitorProbes
* ExtendedDTraceProbes

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.


```
    ((cite: hotspot/src/share/vm/services/dtraceAttacher.hpp))
     private:
      // disable one or more probes - OR above constants
      static void disable_dprobes(int probe_types);
    
     public:
      // enable one or more probes - OR above constants
      static void enable_dprobes(int probe_types);
      // all clients detached, do any clean-up
      static void detach_all_clients();
      // set ExtendedDTraceProbes flag
      static void set_extended_dprobes(bool value);
      // set DTraceMonitorProbes flag
      static void set_monitor_dprobes(bool value);
```

#### 参考(for your information): DTrace::disable_dprobes()
See: [here](no31150YYb.html) for details
#### 参考(for your information): DTrace::enable_dprobes()
See: [here](no2114N1E.html) for details
#### 参考(for your information): DTrace::detach_all_clients()
See: [here](no31150lih.html) for details
#### 参考(for your information): DTrace::set_extended_dprobes()
See: [here](no31150ysn.html) for details
#### 参考(for your information): DTrace::set_monitor_dprobes()
See: [here](no31150_2t.html) for details



### 詳細(Details)
See: [here](../doxygen/classDTrace.html) for details

---
## <a name="nougXBkwDc" id="nougXBkwDc">VM_DeoptimizeTheWorld</a>

### 概要(Summary)
DTrace クラス内で使用される補助クラス.
Solaris 専用のクラス (#ifdef Solaris 時にしか定義されない).

全ての nmethod に対して deoptimize 処理を行う.


```
    ((cite: hotspot/src/share/vm/services/dtraceAttacher.cpp))
    #ifdef SOLARIS
    
    class VM_DeoptimizeTheWorld : public VM_Operation {
```

### 使われ方(Usage)
DTrace::enable_dprobes() 及び DTrace::disable_dprobes() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classVM__DeoptimizeTheWorld.html) for details

---
