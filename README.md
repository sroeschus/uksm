# Ultra Kernel Samepage Merging (aka uksm)
The uksm patches were orginally maintained in [this](https://github.com/dolohow/uksm) repository.
However it looks like the most recent patch has been for 5.17. This repository contains
versions of the uksm patch for more recent versions.

# Why?
I'm currently working at Meta on improving KSM. I'm using the UKSM implementation to
compare it against KSM. I needed a more current implementation of UKSM to compare it
to KSM. I ported this UKSM patch to more current releases. If you use the patches,
please add some reference to this repository.

# Versions
| Version | Changes / Number of hunks         | Testing                         |
|---------|-----------------------------------|---------------------------------|
| 5.18    | 15 hunks changed, difficulty: low | Basic testing                   |
| 5.19    | 10 hunks changed, difficulty: med | Used for long-term perf testing |
| 6.0     | 1 hunk, difficulty: easy          | Basic testing                   |
| 6.1     | 15 hunks, difficulty: med-complex | Full kernel build (1+8 CPU's)   |
| 6.2     | 4 hunks, difficulty: med          | Full kernel build (1+8 CPU's)   |
| 6.3     | 8 hunks, difficulty: med          | Full kernel build (1+8 CPU's)   |
| 6.4     | 11 hunks, difficulty: easy        | Full kernel build (1+8 CPU's)   |
| 6.5     | 2 hunks, difficulty: easy         | Full kernel build (1+8 CPU's)   |
| 6.6     | 9 hunks, difficulty: easy         | Full kernel build (1+8 CPU's)   |
| 6.7     | 0 hunks, difficulty: easy         | Full kernel build (1+8 CPU's)   |
| 6.8     | 20 hunks, difficulty: med-complex | Full kernel build (1+8 CPU's)   |

The Changes column contains the changes compared to the previous version. The
testing column specifies how much testing has been done for this patch. Basic testing
means that the kernel has been started and used for 30 - 60 minutes.

On the later kernels I use a modified version of the test program ksm_tests.c.
In addition I run a kernel build with one and eight CPU's to test the kernel.

# Disclaimer
Use the patches at your own risk. The patches have been prepared for the kernels
as they are released on kernel.org. Linux distribution kernels deviate considerably
from upstream. The patches might apply cleanly or not.

# Applying the patch
How to apply the patch? You need to have a source with the kernel sources, then
the patch can applied with the patch command. For instance:

patch -p1 < ~/uksm/v6.x/uksm-6.0.patch

This should output something like this:
```
patch -p1 < ~/uksm/v6.x/uksm-v6.0.patch
patching file Documentation/vm/uksm.txt
patching file fs/exec.c
patching file fs/proc/meminfo.c
patching file include/linux/ksm.h
patching file include/linux/mm_types.h
patching file include/linux/mmzone.h
patching file include/linux/pgtable.h
patching file include/linux/sradix-tree.h
patching file include/linux/uksm.h
patching file kernel/fork.c
patching file lib/Makefile
patching file lib/sradix-tree.c
patching file mm/Kconfig
patching file mm/Makefile
patching file mm/ksm.c
patching file mm/memory.c
patching file mm/mmap.c
patching file mm/uksm.c
patching file mm/vmstat.c
```

If all goes well all hunks are applied. Just compile and install the kernel (this assumes you
have a valid kernel config file).

```
make clean
make -j$(nproc)
```

# Changes
## 5.18
- get_mergeable_page_lock_mmap(): needs to use folio
- write_protext_page(): needs to replace page_vma_mapped_walk with DEFINE_FOLIO_VMA_WALK
- replace_page(): whitespace fix
- replace_page(): page_remove_ramp needs an additional parameter 
- try_to_merge_with_uksm_page(): munlock_vma_page() and mlock_vma_page() is no longer needed
- try_to_merge_two_pages(): munlock_vma_page() and mlock_vma_page() is no longer needed
- try_merge_rmap_item(): page_vma_mapped_walk replaced with DEFINE_FOLIO_MAP_WALK 
- try_merge_rmap_item(): page_remove_rmap() needs additional parameter
- try_merge_with_stable(): vma1 and vma2 variables no longer needed
- try_merge_with_stable(): mlock_vma_page() no longer needed
- rmap_walk_ksm(): second argument changed to const
- rmap_walk_ksm(): VM_BUG_ON_PAGE needs to use folios
- rmap_walk_ksm(): rmap_one() and rmap_done() expects folio parameter
- ksm_might_need_to_copy(): anon_vma needs to use folio

## 5.19
- page_stable_node(): removed
- set_page_stable_node(): removed
- zap_pte_range(): hunk for call to uksm_map_zero_page() does not cleanly apply
- rmap_walk_ksm(): remove const from second argument in function signature
- write_protect_page(): add anon_exclusive handling
- replace_page(): last parameter to page_add_anon_rmap() must be RMAP_NONE
- try_merge_rmap_item: last parameter to page_add_anon_rmap() must be RMAP_NONE
- rmap_walk_ksm(): add anon_vma_trylock_read() handling

## 6.0
- lib/Makefile: Manually apply one hunk

## 6.1
Considerable changes in memory management due to the introduction of the maple tree.
- include/linux/uksm.h: renamed uksm_vma_add_new to uksm_add_vma_new()
- kernel/fork.c: vm_area_free() no longer using __vma_link_rb()
- kernel/fork.c: dup_mmap: using mas, requires additional call to uksm_add_vma_new()
- lib/Makefile: Manually apply one hunk
- mm/map.c: vma_expand() new calls to uksm_remove_vma() and uksm_add_vma_new()
- mm/map.c: Changes due to maple_tree 
- mm/map.c: __split_vma(): moving uksm_add_vma_new() call
- mm/map.c: mmap_region(): added call to uksm_add_vma_new()
- mm/map.c: do_brk_flags(): added call to uksm_remove_vma()
- mm/map.c: exit_mmap(): removed invalidation
- mm/uksm.c: Added BUG_ON to uksm_remove_vma() to catch bugs
- mm/uksm.c: Use folio API's in replace_page()
- mm/uksm.c: 3 hunks, replace prandom_u32() with get_random_u32()

## 6.2
- mm/uksm.c: replaced write_page() with updated api's
- mm/uksm.c: replaced replace_page() with updated api's
- fs/exec.c: fixed header conflict
- mm/memory.c: fixed conflict with fast counters
- mm/memory.c: fixed conflict with new copy_mc_user_highpage
- mm/memory.c: fixed conflict with fast counters

## 6.3
- A couple of changes to address the changes vma API's
- mm/memory.c: fixed conflict in copy_present_pte()
- mm/mmap.c: fixed 2 conflicts in vma_expand(), considerable changes to vma functions
- mm/mmap.c: fixed 2 conflicts in vma_shrink(), this is a new function
- mm/mmap.c: fixed conflict in __split_vma()
- mm/mmap.c: fixed two conflicts in do_brk_flags()

## 6.4
- Added some new empty functions to uksm.c to silence linker
- fs/exec.c: fixed conflict in include
- include/linux/ksm.h: fixed 3 conflicts
- kernel/fork.c: conflict in vm_area_dup()
- kernel/fork.c: conflict in _vm_area_free()
- mm/mmap.c: conflict in remove_vma()
- mm/mmap.c: conflict in vma_expand()
- mm/mmap.c: conflict in vma_merge()
- mm/mmap.c: 2 conflicts in do_brk_flags()

## 6.5
- mm/uksm.c: use ptep_get() api in break_ksm_pmd_entry()
- mm/uksm.c: use read-only/write pagewalks in break_ksm() and add new parameter
- mm/uksm.c: add new parameter to unmerge_ksm_pages()
- mm/uksm.c: use ptep_get() in write_protect_page()
- mm/uksm.c: use pmd_get_lockless() in replace_page()
- mm/uksm.c: use ptep_get in replace_page()
- mm/uksm.c: add check for poison page in ksm_might_need_to_copy()
- include/linux/mmzone.h: Fixed conflict in zone_stat_item
- mm/mmap.c: Conflict in __split_vma()

## 6.6
- mm/ksm.h: move page_stable_node from ksm.h to ksm.c/uksm.c
- mm/ksm.h: move set_page_stable_node from ksm.h to ksm.c/uksm.c
- mm/ksm.h: define ksm_might_unmap_zero_page for CONFIG_UKSM
- mm/memory.c: Conflict in zap_pte_rang()
- mm/memory.c: Conflict in wp_page_copy()
- mm/mmap.c: 2 conflicts in vma_expand()
- mm/mmap.c: 3 conflicts in vma_shrink()
- mm/mmap.c: Conflict in __split_vma()
- mm/mmap.c: Conflict in do_brk_flags()

## 6.7
- No conflicts

## 6.8
- fs/exec.c: Conflicts with includes
- include/linux/ksm.h: conflict with ksm_might_unmap_zero_page()
- include/linux/ksm.h: also needed ksm_might_unmap_zero_page() in uksm.c
- mm/memory.c: Conflict in wp_page_copy()
- mm/uksm.c: in write_protect_page() replace page_try_share_anon_rmap() with folio_try_share_anon_rmap_pte()
- mm/uksm.c: use folio in replace_page()
- mm/uksm.c: in replace_page() replace page_add_anon_rmap() with folio_add_anon_rmap_pte()
- mm/uksm.c: in replace_page() replace page_remove_rmap() with folio_remove_rmap_pte()
- mm/uksm.c: many changes in ksm_might_need_to_copy() to use folio api's
