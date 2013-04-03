---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiManageCapabilities.cpp

### 名前(function name)
```
void JvmtiManageCapabilities::update() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jvmtiCapabilities avail;
	
  {- -------------------------------------------
  (1) 潜在的に利用可能な全ての capabilities を計算する.
      ---------------------------------------- -}

	  // all capabilities
	  either(&always_capabilities, &always_solo_capabilities, &avail);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (以下の interp_events は, interp_only_mode に関連した capabilities が取得されているかどうかを示す)
      ---------------------------------------- -}

	  bool interp_events =
	    avail.can_generate_field_access_events ||
	    avail.can_generate_field_modification_events ||
	    avail.can_generate_single_step_events ||
	    avail.can_generate_frame_pop_events ||
	    avail.can_generate_method_entry_events ||
	    avail.can_generate_method_exit_events;

  {- -------------------------------------------
  (1) もし以下のようなイベントが利用可能であれば, UseFastEmptyMethods や UseFastAccessorMethods は false にする.
      (これらは generate_empty_entry() や generate_accessor_entry() に関係)
      ---------------------------------------- -}

	  bool enter_all_methods =
	    interp_events ||
	    avail.can_generate_breakpoint_events;
	  if (enter_all_methods) {
	    // Disable these when tracking the bytecodes
	    UseFastEmptyMethods = false;
	    UseFastAccessorMethods = false;
	  }
	
  {- -------------------------------------------
  (1) breakpoint に対応する場合は, RewriteFrequentPairs はオフにする
      ---------------------------------------- -}

	  if (avail.can_generate_breakpoint_events) {
	    RewriteFrequentPairs = false;
	  }
	
  {- -------------------------------------------
  (1) OnLoad phase で can_redefine_classes や can_retransform_class がセットされていたら,
      JvmtiExport::_all_dependencies_are_recorded を true にして
      JIT compiler に全ての dependencies 情報を記録させるようにする.
    
      (JvmtiExport::_all_dependencies_are_recorded は,
      全ての dependencies 情報が残っているのかどうかを RedefineClasses 時に知るためのフラグ)
      ---------------------------------------- -}

	  // If can_redefine_classes is enabled in the onload phase then we know that the
	  // dependency information recorded by the compiler is complete.
	  if ((avail.can_redefine_classes || avail.can_retransform_classes) &&
	      JvmtiEnv::get_phase() == JVMTI_PHASE_ONLOAD) {
	    JvmtiExport::set_all_dependencies_are_recorded(true);
	  }
	
  {- -------------------------------------------
  (1) JvmtiExport::set_can_*() を呼び出して, 対応する JvmtiExport::can_*() の値をセットする.
      ---------------------------------------- -}

	  JvmtiExport::set_can_get_source_debug_extension(avail.can_get_source_debug_extension);
	  JvmtiExport::set_can_maintain_original_method_order(avail.can_maintain_original_method_order);
	  JvmtiExport::set_can_post_interpreter_events(interp_events);
	  JvmtiExport::set_can_hotswap_or_post_breakpoint(
	    avail.can_generate_breakpoint_events ||
	    avail.can_redefine_classes ||
	    avail.can_retransform_classes);
	  JvmtiExport::set_can_modify_any_class(
	    avail.can_generate_breakpoint_events ||
	    avail.can_generate_all_class_hook_events);
	  JvmtiExport::set_can_walk_any_space(
	    avail.can_tag_objects);   // disable sharing in onload phase
	  // This controls whether the compilers keep extra locals live to
	  // improve the debugging experience so only set them if the selected
	  // capabilities look like a debugger.
	  JvmtiExport::set_can_access_local_variables(
	    avail.can_access_local_variables ||
	    avail.can_generate_breakpoint_events ||
	    avail.can_generate_frame_pop_events);
	  JvmtiExport::set_can_post_on_exceptions(
	    avail.can_generate_exception_events ||
	    avail.can_generate_frame_pop_events ||
	    avail.can_generate_method_exit_events);
	  JvmtiExport::set_can_post_breakpoint(avail.can_generate_breakpoint_events);
	  JvmtiExport::set_can_post_field_access(avail.can_generate_field_access_events);
	  JvmtiExport::set_can_post_field_modification(avail.can_generate_field_modification_events);
	  JvmtiExport::set_can_post_method_entry(avail.can_generate_method_entry_events);
	  JvmtiExport::set_can_post_method_exit(avail.can_generate_method_exit_events ||
	                                        avail.can_generate_frame_pop_events);
	  JvmtiExport::set_can_pop_frame(avail.can_pop_frame);
	  JvmtiExport::set_can_force_early_return(avail.can_force_early_return);
	  JvmtiExport::set_should_clean_up_heap_objects(avail.can_generate_breakpoint_events);
	}
	
```


