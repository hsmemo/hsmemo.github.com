---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/biasedLocking.cpp

### 名前(function name)
```
  virtual void task() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) VM_EnableBiasedLocking により biased locking を有効化する.
      ---------------------------------------- -}

	    // Use async VM operation to avoid blocking the Watcher thread.
	    // VM Thread will free C heap storage.
	    VM_EnableBiasedLocking *op = new VM_EnableBiasedLocking(true);
	    VMThread::execute(op);
	
  {- -------------------------------------------
  (1) biased locking の有効化処理を行うのは1回でいいので, 終わったら自分自身を消去する.
      ---------------------------------------- -}

	    // Reclaim our storage and disenroll ourself
	    delete this;
	  }
	
```


