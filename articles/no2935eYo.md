---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnvBase.cpp
### 説明(description)

```
// Based on the stack information in the linked list, allocate memory
// block to return and fill it from the info in the linked list.
```

### 名前(function name)
```
void
VM_GetMultipleStackTraces::allocate_and_fill_stacks(jint thread_count) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // do I need to worry about alignment issues?
	  jlong alloc_size =  thread_count       * sizeof(jvmtiStackInfo)
	                    + _frame_count_total * sizeof(jvmtiFrameInfo);
	  env()->allocate(alloc_size, (unsigned char **)&_stack_info);
	
	  // pointers to move through the newly allocated space as it is filled in
	  jvmtiStackInfo *si = _stack_info + thread_count;      // bottom of stack info
	  jvmtiFrameInfo *fi = (jvmtiFrameInfo *)si;            // is the top of frame info
	
	  // copy information in resource area into allocated buffer
	  // insert stack info backwards since linked list is backwards
	  // insert frame info forwards
	  // walk the StackInfoNodes
	  for (struct StackInfoNode *sin = head(); sin != NULL; sin = sin->next) {
	    jint frame_count = sin->info.frame_count;
	    size_t frames_size = frame_count * sizeof(jvmtiFrameInfo);
	    --si;
	    memcpy(si, &(sin->info), sizeof(jvmtiStackInfo));
	    if (frames_size == 0) {
	      si->frame_buffer = NULL;
	    } else {
	      memcpy(fi, sin->info.frame_buffer, frames_size);
	      si->frame_buffer = fi;  // point to the new allocated copy of the frames
	      fi += frame_count;
	    }
	  }
	  assert(si == _stack_info, "the last copied stack info must be the first record");
	  assert((unsigned char *)fi == ((unsigned char *)_stack_info) + alloc_size,
	         "the last copied frame info must be the last record");
	}
	
```


