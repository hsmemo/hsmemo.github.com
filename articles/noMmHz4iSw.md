---
layout: default
title: Serviceability 機能 ： OS のプロファイラ機能との連携 ： OProfile
---
[Up](no3ZCVJmJ2.html) [Top](../index.html)

#### Serviceability 機能 ： OS のプロファイラ機能との連携 ： OProfile

--- 
#Under Construction

## 概要(Summary)
(#Under Construction)

## 備考(Notes)
この機能を使用するには UseOprofile オプションをセットする必要がある.

## 処理の流れ (概要)(Execution Flows : Summary)
```
(See: )
-> CodeHeap::reserve()
   -> CodeHeap::on_code_mapping()
      -> linux_wrap_code()

(See: )
-> CodeHeap::expand_by()
   -> CodeHeap::on_code_mapping()
      -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)







