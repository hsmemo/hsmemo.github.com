---
layout: default
title: FileMapInfo クラス 
---
[Top](../index.html)

#### FileMapInfo クラス 



---
## <a name="no6pXrwfIa" id="no6pXrwfIa">FileMapInfo</a>

### 概要(Summary)
Class Data Sharing (CDS) 機能用の補助クラス (See: [here](no2114Sn1.html) for details).

shared archive ファイル (クラス情報をダンプしたファイル) を操作するためのクラス.


```
    ((cite: hotspot/src/share/vm/memory/filemap.hpp))
    class FileMapInfo : public CHeapObj {
```

### 使われ方(Usage)
以下の両方の処理で使用されている.

* メモリの内容をファイルにダンプする処理
* 出力したファイルを読み込む処理(= ファイルをメモリ空間にマッピングする処理)

#### 参考(for your information): メモリの内容をファイルにダンプする処理 (VM_PopulateDumpSharedSpace::doit())

```
    ((cite: hotspot/src/share/vm/memory/dump.cpp))
      void doit() {
    ...
        // Create and write the archive file that maps the shared spaces.
    
        FileMapInfo* mapinfo = new FileMapInfo();
        mapinfo->populate_header(gch->gen_policy()->max_alignment());
    
        // Pass 1 - update file offsets in header.
        mapinfo->write_header();
        mapinfo->write_space(CompactingPermGenGen::ro, _ro_space, true);
        _ro_space->set_saved_mark();
        mapinfo->write_space(CompactingPermGenGen::rw, _rw_space, false);
        _rw_space->set_saved_mark();
        mapinfo->write_region(CompactingPermGenGen::md, _md_vs->low(),
                              pointer_delta(md_top, _md_vs->low(), sizeof(char)),
                              SharedMiscDataSize,
                              false, false);
        mapinfo->write_region(CompactingPermGenGen::mc, _mc_vs->low(),
                              pointer_delta(mc_top, _mc_vs->low(), sizeof(char)),
                              SharedMiscCodeSize,
                              true, true);
    
        // Pass 2 - write data.
        mapinfo->open_for_write();
        mapinfo->write_header();
        mapinfo->write_space(CompactingPermGenGen::ro, _ro_space, true);
        mapinfo->write_space(CompactingPermGenGen::rw, _rw_space, false);
        mapinfo->write_region(CompactingPermGenGen::md, _md_vs->low(),
                              pointer_delta(md_top, _md_vs->low(), sizeof(char)),
                              SharedMiscDataSize,
                              false, false);
        mapinfo->write_region(CompactingPermGenGen::mc, _mc_vs->low(),
                              pointer_delta(mc_top, _mc_vs->low(), sizeof(char)),
                              SharedMiscCodeSize,
                              true, true);
        mapinfo->close();
```

#### 参考(for your information): ファイルをメモリ空間にマッピングする処理 (universe_init())
FileMapInfo::initialize() でファイルをメモリにマッピングする.

(さらに, FileMapInfo::set_current_info() を呼んで, 
 マッピングした内容を表す FileMapInfo オブジェクトを
 FileMapInfo クラスの static フィールドに記録している)

```
    ((cite: hotspot/src/share/vm/memory/universe.cpp))
    jint universe_init() {
    ...
      FileMapInfo* mapinfo = NULL;
      if (UseSharedSpaces) {
        mapinfo = NEW_C_HEAP_OBJ(FileMapInfo);
        memset(mapinfo, 0, sizeof(FileMapInfo));
    
        // Open the shared archive file, read and validate the header. If
        // initialization files, shared spaces [UseSharedSpaces] are
        // disabled and the file is closed.
    
        if (mapinfo->initialize()) {
          FileMapInfo::set_current_info(mapinfo);
```

### 内部構造(Internal structure)
FileMapInfo クラス内には FileMapInfo オブジェクトを格納しておく static フィールドが 1つあり,
FileMapInfo::current_info() でマッピングしたファイルの情報にアクセスできるようになっている.




### 詳細(Details)
See: [here](../doxygen/classFileMapInfo.html) for details

---
