---
layout: default
title: JDK_Version クラス及び ExitProc クラス (JDK_Version, ExitProc)
---
[Top](../index.html)

#### JDK_Version クラス及び ExitProc クラス (JDK_Version, ExitProc)

これらは, 標準クラスライブラリ(JDK)のバージョン情報, 及び HotSpot の終了時処理を管理するためのクラス.

(なおここでの JDK とは標準クラスライブラリ(libjava.so)のこと)


### クラス一覧(class list)

  * [JDK_Version](#nolmMYYlsn)
  * [ExitProc](#nokOJtbRMf)


---
## <a name="nolmMYYlsn" id="nolmMYYlsn">JDK_Version</a>

### 概要(Summary)
使用している JDK のバージョンを管理するクラス
(なおここでの JDK とは標準クラスライブラリ(libjava.so)のこと).

JDK のバージョンを検出し, その情報を格納しておく役割がある.

(なおコメントによると, 
 JDK 6 以降では libjava.so が GetVersion (正確には JDK_GetVersionInfo0 ?) 
 という関数を提供してくれているため簡単に検出できるが, 
 それ以前のバージョンだと少し難しい, 
 とのこと.
 
 このため, 
 JDK 6 以降のものについては JDK_Version::initialize() が呼ばれた段階で完全に分かるようになるが, 
 JDK 5 以前のものでは JDK_Version::fully_initialize() が呼ばれるまでは詳細は分からない
 (JDK 5 以前だということしか分からない) 状態になっている.)


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.hpp))
    /**
     * Discovering the JDK_Version during initialization is tricky when the
     * running JDK is less than JDK6.  For JDK6 and greater, a "GetVersion"
     * function exists in libjava.so and we simply call it during the
     * 'initialize()' call to find the version.  For JDKs with version < 6, no
     * such call exists and we have to probe the JDK in order to determine
     * the exact version.  This probing cannot happen during late in
     * the VM initialization process so there's a period of time during
     * initialization when we don't know anything about the JDK version other than
     * that it less than version 6.  This is the "partially initialized" time,
     * when we can answer only certain version queries (such as, is the JDK
     * version greater than 5?  Answer: no).  Once the JDK probing occurs, we
     * know the version and are considered fully initialized.
     */
    class JDK_Version VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
JDK_Version クラスの _current フィールド (static フィールド) に(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.hpp))
      static JDK_Version _current;
```

#### 生成箇所(where its instances are created)
(JDK_Version クラスの _current フィールドは, ポインタ型ではなく実体なので,
初期段階で自動的に生成される)

#### 初期化箇所(where its instances are initialized)
Threads::create_vm() 内で(のみ)初期化されている.

<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; JDK_Version_init()
      -&gt; JDK_Version::initialize()
         (libjava.so が GetVersion() を提供してくれていれば, ここで検出して設定する)
   -&gt; init_globals()
      -&gt; universe2_init()
         -&gt; Universe::genesis()
            (libjava.so が GetVersion() を提供してくれていなければ, ここでバージョンを推定して設定する)
            -&gt; JDK_Version::fully_initialize()
</pre></div>

#### 参考(for your information): JDK_Version_init()
See: [here](no17119E8A.html) for details
#### 参考(for your information): JDK_Version::initialize()
See: [here](no17119eQN.html) for details
#### 参考(for your information): JDK_Version::fully_initialize()
See: [here](no17119RGH.html) for details
### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.hpp))
      // In this class, we promote the minor version of release to be the
      // major version for releases >= 5 in anticipation of the JDK doing the
      // same thing.  For example, we represent "1.5.0" as major version 5 (we
      // drop the leading 1 and use 5 as the 'major').
    
      uint8_t _major;
      uint8_t _minor;
      uint8_t _micro;
      uint8_t _update;
      uint8_t _special;
      uint8_t _build;
    
      // If partially initialized, the above fields are invalid and we know
      // that we're less than major version 6.
      bool _partially_initialized;
    
      bool _thread_park_blocker;
      bool _post_vm_init_hook_enabled;
```




### 詳細(Details)
See: [here](../doxygen/classJDK__Version.html) for details

---
## <a name="nokOJtbRMf" id="nokOJtbRMf">ExitProc</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらないような...)


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
    class ExitProc : public CHeapObj {
```

(java.io.File.deleteOnExit() から使われると書いてあるが, 実際には使われていない.
 File.deleteOnExit() は java.lang.Shutdown.shutdown() から呼び出される shutdown hook で実現されている)


```cpp
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
    //      > run VM level shutdown hooks (they are registered through JVM_OnExit(),
    //        currently the only user of this mechanism is File.deleteOnExit())
```

(<= どうやら, File.deleteOnExit() は, 当初は ExitProc で実装されていた模様.
しかし, Java 1.3 で shutdown hook が導入され, その後 File.deleteOnExit() も shutdown hook ベースの実装に修正された模様.
[参考URL](http://bugs.sun.com/view_bug.do?bug_id=4809375))


### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 実行したい関数をコンストラクタ引数として ExitProc オブジェクトを生成する.
2. 生成した ExitProc オブジェクトに対して ExitProc::evaluate() を呼ぶと, コンストラクタで渡した関数が実行される.

#### インスタンスの格納場所(where its instances are stored)
exit_procs という大域変数に(のみ)格納されている.

(正確には, このフィールドは ExitProc の線形リストを格納するフィールド.
ExitProc オブジェクトは _next フィールドで次の ExitProc オブジェクトを指せる構造になっている.
生成した ExitProc オブジェクトは全てこの線形リスト内に格納されている)

#### 生成箇所(where its instances are created)
register_on_exit_function() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

(が, この JVM_OnExit() は使われていないような...?? #TODO)

<div class="flow-abst"><pre>
JVM_OnExit()
-&gt; register_on_exit_function()
</pre></div>

#### 使用箇所(where its instances are used)
before_exit() 内で(のみ)使用されている.

(exit_procs 内の全ての ExitProc が取り出され, ExitProc::evaluate() が呼び出される)




### 詳細(Details)
See: [here](../doxygen/classExitProc.html) for details

---
