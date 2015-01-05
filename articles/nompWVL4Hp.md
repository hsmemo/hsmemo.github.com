---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： 起動時処理 (JVMTI エージェントのロード & Agent_OnLoad()/JVM_Onload() の呼び出し)
---
[Up](no1sX8Q67Q.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： 起動時処理 (JVMTI エージェントのロード & Agent_OnLoad()/JVM_Onload() の呼び出し)

--- 
## 概要(Summary)
JVMTI エージェントのロード処理 (及び Agent_OnLoad()/JVM_Onload() の呼び出し処理) は, 
HotSpot の起動時(初期化時)に行われる.

なお, -agentlib オプションや -agentpath オプションで指定されたエージェントと, 
-Xrun オプションで指定されたエージェントは若干扱いが異なる.

-agentlib オプションや -agentpath オプションで指定されたエージェントは, 以下のように処理される.
  
  1. まず, Arguments::parse_each_vm_init_arg() 関数の中でオプションがパースされる.

     (より正確には, そこから呼び出される Arguments::add_init_agent() で処理が行われる)

  2. 次に, Threads::create_vm_init_agents() で実際にエージェントがロードされ, 
     それぞれの Agent_OnLoad() 関数が呼び出される.
     
-Xrun オプションで指定されたエージェントは, 以下のように処理される.
  
  1. まず, Arguments::parse_each_vm_init_arg() 関数の中でオプションがパースされる.

     (より正確には, そこから呼び出される Arguments::add_init_library() で処理が行われる)

     なお, JVM_OnLoad() が定義されていなければ -agentlib や -agentpath と同じ形式に変換される.
     この場合, 残りの処理は -agentlib や -agentpath の場合と同じになる.

  2. JVM_OnLoad() が定義されている場合は, 
     起動時処理の最後の方で JVM_Onload() の呼び出しが行われる.


## 備考(Notes)
-agentlib オプションや -agentpath オプションで指定されたエージェントは
Arguments::_agentList フィールドの AgentLibraryList で管理されている.

-Xrun オプションで指定されたエージェントは, 
JVM_OnLoad() が定義されていれば Arguments::_libraryList フィールドの 
AgentLibraryList で管理されている.
定義されていなければ, 一旦 Arguments::_libraryList フィールドに格納された後, 
Arguments::_agentList フィールドに移される.

## 備考(Notes)
<http://java.sun.com/developer/technicalArticles/Programming/jvmpitransition/>

「JVMPI/JVMDI と JVMTI の両対応のエージェントを作れるように,
-Xrun でも Agent_OnLoad() が定義されていれば JVM_OnLoad() を無視して -agentpath と同様に扱い,
そうでなければ 1.4 以前と同様に JVM_OnLoad() を呼び出す」
と書かれているが, 若干挙動が違うような...(#TODO)


## 処理の流れ (概要)(Execution Flows : Summary)
(なお, Arguments::parse_each_vm_init_arg() は 
Arguments::parse_vm_init_args() 内の 3箇所で呼び出されている)

<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; (1) 関連するオプション(-Xrun, -agentlib, -agentpath)のパース処理を行う
          -&gt; Arguments::parse()
             -&gt; Arguments::parse_vm_init_args()
                -&gt; Arguments::parse_each_vm_init_arg()
                   -&gt; Arguments::add_init_library()  (&lt;= -Xrun オプションの処理)
                      -&gt; AgentLibraryList::add()
                   -&gt; Arguments::add_init_agent()    (&lt;= -agentlib オプション及び -agentpath オプションの処理)
                      -&gt; AgentLibraryList::add()
                -&gt; Arguments::parse_java_options_environment_variable()
                   -&gt; Arguments::parse_options_environment_variable()
                      -&gt; Arguments::parse_each_vm_init_arg()
                         -&gt; (同上)
                -&gt; Arguments::parse_java_tool_options_environment_variable()
                   -&gt; Arguments::parse_options_environment_variable()
                      -&gt; (同上)

      (1) -Xrun オプションの値を -agentlib/-agentpath の形式に変換する (ただし JVM_OnLoad() が定義されているものについては変換しない)
          -&gt; Threads::convert_vm_init_libraries_to_agents()
             -&gt; Arguments::convert_library_to_agent()
                -&gt; AgentLibraryList::remove()
                -&gt; AgentLibraryList::add()

      (1) エージェントをロードし, それぞれの Agent_OnLoad() 関数を呼び出す
          -&gt; Threads::create_vm_init_agents()
             -&gt; lookup_agent_on_load()
                -&gt; lookup_on_load()
                   -&gt; os::dll_build_name()
                   -&gt; os::dll_load()
                   -&gt; os::dll_lookup()
             -&gt; それぞれのエージェント(.soファイル等)の Agent_OnLoad() 関数を呼び出す

      (1) -Xrun オプションで指定されたエージェントで JVM_OnLoad() が定義されているものについて, それぞれの JVM_OnLoad() 関数を呼び出す
          -&gt; Threads::create_vm_init_libraries()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### Arguments::add_init_library()
See: [here](no32740JcL.html) for details
### AgentLibraryList::add()
See: [here](no32740Xgk.html) for details
### Arguments::add_init_agent()
See: [here](no327409Ek.html) for details
### Threads::convert_vm_init_libraries_to_agents()
See: [here](no327409SM.html) for details
### Arguments::convert_library_to_agent()
See: [here](no32740xCZ.html) for details
### Threads::create_vm_init_agents()
See: [here](no32740Yhr.html) for details
### lookup_jvm_on_load()
See: [here](no32740_Ua.html) for details
### lookup_agent_on_load()
See: [here](no32740yKU.html) for details
### lookup_on_load()
See: [here](no32740_bO.html) for details
### os::dll_build_name()  (Linux の場合)
(#Under Construction)

### os::dll_build_name()  (Solaris の場合)
(#Under Construction)

### os::dll_build_name()  (Windows の場合)
(#Under Construction)

### os::dll_load()  (Linux の場合)
See: [here](no17119KQT.html) for details
### os::dll_load()  (Solaris の場合)
See: [here](no17119XaZ.html) for details
### os::dll_load()  (Windows の場合)
See: [here](no3059djw.html) for details
### os::dll_lookup()  (Linux の場合)
See: [here](no17119WZS.html) for details
### os::dll_lookup()  (Solaris の場合)
See: [here](no17119jjY.html) for details
### os::dll_lookup()  (Windows の場合)
See: [here](no3059qt2.html) for details
### Threads::create_vm_init_libraries()
See: [here](no32740AdV.html) for details






