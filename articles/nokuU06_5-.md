---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ClassLoad イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ClassLoad イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(See: <a href="noIvSV0NZj.html">here</a> for details)
-&gt; SystemDictionary::parse_stream()
   -&gt; JvmtiExport::post_class_load()
      -&gt; (登録されているコールバックを呼び出す)

(See: <a href="noIvSV0NZj.html">here</a> for details)
-&gt; SystemDictionary::resolve_instance_class_or_null()
   -&gt; JvmtiExport::post_class_load()
      -&gt; (同上)

(See: <a href="noIvSV0NZj.html">here</a> for details)
-&gt; SystemDictionary::define_instance_class()
   -&gt; JvmtiExport::post_class_load()
      -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)






