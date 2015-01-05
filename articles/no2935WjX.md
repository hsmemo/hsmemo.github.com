---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ClassFileLoadHook イベントの処理  
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ClassFileLoadHook イベントの処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(See: <a href="no2114rPX.html">here</a> for details)
-&gt; ClassFileParser::parseClassFile()
   -&gt; JvmtiExport::post_class_file_load_hook()
      -&gt; JvmtiClassFileLoadHookPoster::post()
         -&gt; JvmtiClassFileLoadHookPoster::post_all_envs()
            -&gt; JvmtiClassFileLoadHookPoster::post_to_env()
               -&gt; (登録されているコールバックを呼び出す)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)







