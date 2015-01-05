---
layout: default
title: Serviceability 機能 ： Forte との連携
---
[Up](noOQc_VTg2.html) [Top](../index.html)

#### Serviceability 機能 ： Forte との連携

--- 
## 概要(Summary)
動的に生成したコードは, Forte::register_stub() によってパフォーマンスアナライザに登録される
(といっても collector_func_load() 関数を呼ぶだけだが...).

また, スレッドのスタックトレースを取得するための関数として AsyncGetCallTrace() が用意されている.
この関数は名前の通り JVMPI の GetCallTrace() 関数 (= JVMTI の GetStackTrace() 関数) の asynchronous 版で, 
パフォーマンスアナライザによって (SIGPROF のハンドラ等から) 呼び出されることを想定している.

### 参考(for your information)
* <http://jeremymanson.blogspot.jp/2007/06/more-about-profiling-with-sigprof.html>
* <http://jeremymanson.blogspot.jp/2007/06/more-thoughts-on-sigprof-jvmti-and.html>
* <http://jeremymanson.blogspot.jp/2010/07/why-many-profilers-have-serious.html>

## 備考(Notes)
collector_func_load() が存在しない環境では呼び出しが NOP になるよう, マクロが定義されている.


```cpp
    ((cite: hotspot/src/share/vm/prims/forte.cpp))
    #ifndef _WINDOWS
    // Support for the Forte(TM) Peformance Tools collector.
    //
    // The method prototype is derived from libcollector.h. For more
    // information, please see the libcollect man page.
    
    // Method to let libcollector know about a dynamically loaded function.
    // Because it is weakly bound, the calls become NOP's when the library
    // isn't present.
    void    collector_func_load(char* name,
                                void* null_argument_1,
                                void* null_argument_2,
                                void *vaddr,
                                int size,
                                int zero_argument,
                                void* null_argument_3);
    #pragma weak collector_func_load
    #define collector_func_load(x0,x1,x2,x3,x4,x5,x6) \
            ( collector_func_load ? collector_func_load(x0,x1,x2,x3,x4,x5,x6),0 : 0 )
    #endif // !_WINDOWS
```

## 処理の流れ (概要)(Execution Flows : Summary)
### Forte::register_stub() の処理
<div class="flow-abst"><pre>
Forte::register_stub()
-&gt; collector_func_load()
</pre></div>

### AsyncGetCallTrace() の処理
<div class="flow-abst"><pre>
AsyncGetCallTrace()
-&gt; forte_fill_call_trace_given_top()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### Forte::register_stub()
See: [here](no52482Gk.html) for details
### AsyncGetCallTrace()
(#Under Construction)
See: [here](no17766NXr.html) for details
### forte_fill_call_trace_given_top()
(#Under Construction)







