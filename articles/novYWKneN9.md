---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： JMX Management Server の起動処理
---
[Up](noGZ5mfSen.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： JMX Management Server の起動処理

--- 
## 概要(Summary)
HotSpot における JMX Management Server (Platform MXBean Server) は
sun.management.Agent クラスの
sun.management.Agent.startAgent() メソッドを呼ぶことで起動できる.

コマンドラインオプションとして -Dcom.sun.management.jmxremote や 
-Dcom.sun.management.snmp を指定することで, 
HotSpot の起動処理中にこのメソッドが呼び出される.

## 備考(Notes)
なお, コマンドラインオプションを指定する以外の方法として, 
management-agent.jar という JPLIS agent (java agent) を Dynamic Attach することでも起動可能.

## 処理の流れ (概要)(Execution Flows : Summary)
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> Management::initialize()
      -> JavaCalls::call_static()
         -> (See: [here](no3059iJu.html) for details)
            -> sun.management.Agent.startAgent()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### Management::initialize()
See: [here](no2114VZN.html) for details
### sun.management.Agent.startAgent()
(#Under Construction)
See: [here](no17766GRw.html) for details
### sun.management.Agent.startAgent(Properties props)
(#Under Construction)







