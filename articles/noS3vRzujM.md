---
layout: default
title: 同期排他処理 ： ロック解放処理 ： slow-path の処理 (2)
---
[Up](noXF2ZIHEZ.html) [Top](../index.html)

#### 同期排他処理 ： ロック解放処理 ： slow-path の処理 (2)

--- 
## 概要(Summary)
fast-path や slow-path 1 が失敗した場合, 
どの処理パスも最終的には ObjectSynchronizer::slow_exit() または ObjectSynchronizer::fast_exit() に到達する.
   
この ObjectSynchronizer::slow_exit()/ObjectSynchronizer::fast_exit() の処理は, 
インタープリタ種別/JIT 種別/CPU 等によらず共通.

なお, 実際の slow-path の処理は ObjectSynchronizer::fast_exit() の方に実装されている.
ObjectSynchronizer::slow_exit() は ObjectSynchronizer::fast_exit() にフォールバックするだけ
(<= 名前からすると逆の方が自然な気もするが...).

## 処理の流れ (概要)(Execution Flows : Summary)
### ObjectSynchronizer::fast_exit() の処理パス
```
-> ObjectSynchronizer::fast_exit()
   -> ObjectSynchronizer::inflate()
   -> ObjectMonitor::exit()
      -> ObjectMonitor::ExitEpilog()
         -> os::PlatformEvent::unpark()
            -> (See: [here](no2114COc.html) for details)
```

### ObjectSynchronizer::slow_exit() の処理パス
```
-> ObjectSynchronizer::slow_exit()
   -> ObjectSynchronizer::fast_exit()
      -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### ObjectSynchronizer::fast_exit()
See: [here](no4230PDV.html) for details
### ObjectSynchronizer::inflate()
See: [here](no4230OJC.html) for details
### ReadStableMark()
(#Under Construction)

### ObjectSynchronizer::omAlloc()
(#Under Construction)

### ObjectSynchronizer::omRelease()
(#Under Construction)

### ObjectMonitor::set_header()
See: [here](no31977QCX.html) for details
### ObjectMonitor::set_owner()
See: [here](no31977dMd.html) for details
### ObjectMonitor::set_object()
See: [here](no31977qWj.html) for details
### oopDesc::release_set_mark()
See: [here](no319773gp.html) for details
### ObjectMonitor::exit()
See: [here](no4230cNb.html) for details
### ObjectMonitor::ExitEpilog()
See: [here](no3059n9T.html) for details
### ObjectSynchronizer::slow_exit()
See: [here](no4230C5O.html) for details






