---
layout: default
title: HotSpot の起動/終了処理 ： HotSpot の終了処理 ： java.lang.System.exit() が呼び出された場合の終了処理  
---
[Up](no28916GoL.html) [Top](../index.html)

#### HotSpot の起動/終了処理 ： HotSpot の終了処理 ： java.lang.System.exit() が呼び出された場合の終了処理  

--- 
## 概要(Summary)
これは Java のプログラム中で java.lang.System.exit() が呼び出された場合の処理.

Java のレベルでは, 最終的には java.lang.Shutdown オブジェクトに行き着き, 以下の処理が実行される.

1. java.lang.Shutdown.sequence() で, shutdown hook を実行する.
2. java.lang.Shutdown.halt() で, 終了処理を呼び出す.

(#Under Construction)

最終的には exit() システムコールを呼ぶことで終了する.

## 処理の流れ (概要)(Execution Flows : Summary)
```
java.lang.System.exit()
-> java.lang.Runtime.exit()
   -> java.lang.Shutdown.exit()
      -> java.lang.Shutdown.sequence()
      -> java.lang.Shutdown.halt()           (← ここまでが Java の世界. 以下は HotSpot 内部の世界)
         -> Java_java_lang_Shutdown_halt0()
            -> JVM_Halt()
               -> before_exit()
               -> vm_exit()
                  -> VM_Exit::doit()         (← ここは飛ばしていきなり vm_direct_exit() に行くパスもある)
                     -> vm_direct_exit()
                        -> exit()
```

## 備考(Notes)
JVM_Exit() という関数も存在し, この中でも before_exit() や vm_exit() が呼ばれている.
が, JVM_Exit() 自体がどこからも呼ばれていない模様...

(コメントを見るかぎりでは java.lang.System() 用の CVMI 関数のようだが...).

## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.System.exit()
See: [here](no4230Xsn.html) for details
### java.lang.Runtime.exit()
See: [here](no4230k2t.html) for details
### java.lang.Shutdown.exit()
See: [here](no4230xA0.html) for details
### java.lang.Shutdown.sequence()
See: [here](no4230jKD.html) for details
### java.lang.Shutdown.halt()
See: [here](no4230wUJ.html) for details
### Java_java_lang_Shutdown_halt0()
See: [here](no42309eP.html) for details
### JVM_Halt()
See: [here](no4230KpV.html) for details
### before_exit()
See: [here](no31977bKP.html) for details
### vm_exit()
See: [here](no4230Xzb.html) for details
### VM_Exit::doit()
See: [here](no4230xHo.html) for details
### vm_direct_exit()
See: [here](no4230k9h.html) for details

=






