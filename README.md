# uksm
The uksm were orginally maintained in [this](https://github.com/dolohow/uksm) repository.
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

The Changes column contains the changes compared to the previous version. The
testing column specifies how much testing has been done for this patch. Basic testing
means that the kernel has been started and used for 30 - 60 minutes.

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
