---
layout: default
title: Class 情報関係の初期化処理  
---
[Up](no38NSe1ks.html) [Top](../index.html)

#### Class 情報関係の初期化処理  

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; init_globals()
      -&gt; classLoader_init()
         -&gt; ClassLoader::initialize()
            -&gt; (1) libzip をロードする
                   -&gt; ClassLoader::load_zip_library()
               (1) ClassPathEntry オブジェクトの初期化を行う.
                   (system class path (boot class path) に対応する
                   ClassPathEntry オブジェクトを生成し, ClassLoader 内に登録する)
                   -&gt; ClassLoader::setup_bootstrap_search_path()
                      -&gt; ClassLoader::update_class_path_entry_list()
                         -&gt; ClassLoader::create_class_path_entry()
                            -&gt; LazyClassPathEntry::LazyClassPathEntry()
                               or ClassPathZipEntry::ClassPathZipEntry()
                               or ClassPathDirEntry::ClassPathDirEntry()
                         -&gt; ClassLoader::add_to_list()
               (1) 遅延ロードの必要があれば (= LazyBootClassLoader オプションが指定されていれば)
                   MetaIndex の初期化を行う (See: MetaIndex)
                   -&gt; ClassLoader::setup_meta_index()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### classLoader_init()
See: [here](no2747ABZ.html) for details
### ClassLoader::initialize()
See: [here](no2747z2S.html) for details
### ClassLoader::load_zip_library()
See: [here](no7517YzS.html) for details
### ClassLoader::setup_bootstrap_search_path()
See: [here](no7517O6J.html) for details
### ClassLoader::update_class_path_entry_list()
See: [here](no75172gX.html) for details
### ClassLoader::create_class_path_entry()
See: [here](no2747mzA.html) for details
### ClassLoader::add_to_list()
See: [here](no75174il.html) for details
### ClassLoader::setup_meta_index()
(#Under Construction)






