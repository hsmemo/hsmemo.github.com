---
layout: default
title: Serviceability 機能 ： jvmstat 機能 (PerfData 機能) 
---
[Up](noOQc_VTg2.html) [Top](../index.html)

#### Serviceability 機能 ： jvmstat 機能 (PerfData 機能) 

--- 
## 参考(for your information)
* <http://openjdk.java.net/groups/hotspot/docs/Serviceability.html>
  
  ("HotSpot Jvmstat Performance Counters" 以降に説明あり)

## 概要(Summary)
(#Under Construction)

HotSpot 内の統計情報を出力する機能. 
なお, 非公式のインターフェースなので使用には注意が必要. "java." 以外で始まるものは特に非公式度が高い.

内部的には PerfData クラスを用いて情報が蓄積される (See: [here](noOFNhRgni.html) for details).
蓄積された情報は PerfData 用の shared memory file ("hsperfdata" ファイル) 経由で出力されている.

## 備考(Notes)
* この機能を使用するには UsePerfData オプションをセットする必要がある.

* "hsperfdata" ファイルの場所は java.io.tmpdir システムプロパティで指定できる. 

   * Linux や Solaris では, デフォルトでは /tmp/hsperfdata_${user-name}/${vm-id}  になる.
   * Windows 上では TMP や TEMP environment variables で場所を制御可能. 

* データは PerfData 内に格納されているが, 適当なタイミングで "hsperfdata" ファイルに反映される.
  
  この処理は StatSampler クラスが定期的に "hsperfdata" ファイルに書き出すことで行われる.
  実際の書き出し処理は PerfData::sample() (をサブクラスがオーバーライドしたもの) に実装されている.

  * ただし, 変更されない値(constant)の場合は必要ないので行われない.
  * また, PerfString の場合は元々 "hsperfdata" ファイル上に確保している
  * このため, 実際に書き出しの対象になるのは PerfLongVariant だけ.
    
      (PerfLongVariant 以外のクラスでは sample() は何もしないメソッドになっている)
      
      (add_item() の第二引数も参照. 
      PerfLongVariant 以外は false なので PerfDataManager::_sampled に登録されない (= StatSampler の処理対象にならない))

* PerfData オブジェクトをメモリ上に確保する処理は PerfData::create_entry() で行われる.
  ここでは『PerfDataEntry 用のヘッダ + その PerfData の名前 + 実際のデータ』分のメモリが確保される.

  * 先頭に PerfDataEntry 用のヘッダ, 次に名前, 最後に実際のデータが配置される.
   
  * どこからが 名前 でどこからがデータかは, PerfDataEntry のフィールドを見ればわかる
    (PerfDataEntry::name_offset, PerfDataEntry::data_offset).
   
  * そして PerfData からは _pdep フィールドがこの PerfDataEntry の先頭を指している.


## 処理の流れ (概要)(Execution Flows : Summary)
### shared memory file ("hsperfdata" ファイル) の生成処理
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> vm_init_globals()
      -> perfMemory_init()
         -> PerfMemory::initialize()
            -> PerfMemory::create_memory_region()  (<= この処理は OS 毎に異なる)
               -> * Linux の場合:
                    -> * shared file を作らない場合
                         -> 
                       * shared file を作る場合
                         -> create_shared_memory()
                            -> mmap_create_shared()
                               -> (1) shared memory を作るディレクトリ名を計算 (/tmp/hsperfdata_${user-name})
                                      -> get_user_tmp_dir()
      
                                  (2) shared memory file のファイル名を計算 (${vm-id})
                                      -> get_sharedmem_filename()
      
                                  (3) shared memory file を作成
                                      -> create_sharedmem_resources()

                  * Solaris の場合:
                    -> 

                  * Windows の場合:
                    -> 
```

### hsperfdata ファイルの初期化完了を宣言する処理
(PerfDataPrologue::accessible フィールドを true に変更する処理. 
 これが終わると sun.jvmstat.perfdata.monitor.v2_0.isAccessible() が true を返すようになる)

```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> TraceVmCreationTime::end()
      -> Management::record_vm_startup_time()
         -> PerfMemory::set_accessible()
```

### PerfData の生成処理 (& hsperfdata ファイル中に PerfData を確保する処理)
```
PerfLong::PerfLong()
-> PerfData::create_entry()
   -> PerfMemory::alloc()
      -> 

PerfByteArray::PerfByteArray()
-> PerfData::create_entry()
   -> (同上)

PerfDataManager::create_string_constant()
-> PerfString::PerfString()
   -> PerfByteArray::PerfByteArray()
      -> (同上)

PerfDataManager::create_long_constant()
-> PerfLongConstant::PerfLongConstant()
   -> PerfLong::PerfLong()
      -> (同上)

PerfDataManager::create_string_variable(CounterNS ns, const char* name, jint max_length, const char* s, TRAPS)
-> PerfStringVariable::PerfStringVariable()
   -> PerfString::PerfString()
      -> (同上)

PerfDataManager::create_string_variable(CounterNS ns, const char* name, const char *s, TRAPS)
-> PerfDataManager::create_string_variable(CounterNS ns, const char* name, jint max_length, const char* s, TRAPS)
   -> (同上)

PerfDataManager::create_long_variable()  (<= 正確には, 引数違いのものが4種類存在. ただし基本的には全て同じ)
-> PerfLongVariable::PerfLongVariable()
   -> PerfLongVariant::PerfLongVariant()
      -> PerfLong::PerfLong()
         -> (同上)

PerfDataManager::create_long_counter()  (<= 正確には, 引数違いのものが4種類存在. ただし基本的には全て同じ)
-> PerfLongCounter::PerfLongCounter()
   -> PerfLongVariant::PerfLongVariant()
      -> (同上)

PerfDataManager::create_constant()
-> PerfDataManager::create_long_constant()
   -> (同上)

PerfDataManager::create_variable()  (<= 正確には, 引数違いのものが4種類存在. ただし基本的には全て同じ)
-> PerfDataManager::create_long_variable()
   -> (同上)

PerfDataManager::create_counter()  (<= 正確には, 引数違いのものが4種類存在. ただし基本的には全て同じ)
-> PerfDataManager::create_long_counter()
   -> (同上)
```

### PerfData への記録処理


### hsperfdata への値の反映処理
```
PerfLongVariant::PerfLongVariant()
-> PerfLongVariant::sample()

StatSamplerTask::task()
-> StatSampler::collect_sample()
   -> StatSampler::sample_data()
      -> PerfData::sample() (をサブクラスがオーバーライドしたもの)

StatSampler::disengage()
-> StatSampler::sample_data()
   -> (同上)
```


### hsperfdata ファイルを破棄する処理
```
(See: )
-> perfMemory_exit()
   -> PerfDataManager::destroy()
   -> PerfMemory::destroy()
      -> PerfMemory::delete_memory_region()  (<= この処理は OS 毎に異なる)
         -> * Linux の場合:
              -> (1) データをファイルに保存する (<= 必要があれば行う. より具体的に言うと PerfDataSaveToFile 及び PerfDataSaveFile が指定されている場合にのみ行う)
                     -> save_memory_to_file()
      
                 (2) メモリ領域を削除する
                     * shared file を作らなかった場合
                       -> delete_standard_memory()
                     * shared file を作った場合
                       -> delete_shared_memory()

            * Solaris の場合:
              -> 

            * Windows の場合:
              -> 
```


### 他 HotSpot の hsperfdata ファイルにアタッチする処理
```
sun.misc.Perf.attach()
-> Perf_Attach()
   -> PerfMemory::attach()
```

### 他 HotSpot の hsperfdata ファイルからデタッチする処理
```
sun.misc.Perf.detach()
-> Perf_Detach()
   -> PerfMemory::detach()
```

## 処理の流れ (詳細)(Execution Flows : Details)






