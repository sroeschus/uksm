# uksm
The uksm were orginally maintained in [this](https://github.com/dolohow/uksm) repository.
However it looks like the most recent patch has been for 5.17.

# Why?
I'm currently working at Meta on improving KSM. I'm using the UKSM implementation to
compare it against KSM. I needed a more current implementation of UKSM to compare it
to KSM. I ported this UKSM patch to more current releases. If you use the patches,
please add some reference to this repository.

# Versions
| Version | Changes / Number of hunks         | Testing                         |
|---------|-----------------------------------|---------------------------------|
| 5.18    | 15 hunks changed, difficulty: low | Basic testing                   |
| 5.19    | Some minor changes for folios     | Used for long-term perf testing |
|         | Changes to rmap                   |                                 |
|         | Changes to anon vma lock handling |                                 |
| 6.0     | Some minor makefile changes       |                                 |

The Changes column contains the changes compared to the previous version. The
testing column specifies how much testing has been done for this patch. Basic testing
means that the kernel has been started and used for 30 minutes.

# Disclaimer
Use the patches at your own risk. The patches have been prepared for the kernels
as they are released on kernel.org. Linux distribution kernels deviate considerably
from upstream. The patches might apply cleanly or not.

# Applying the patch
How to apply the patch? You need to have a source with the kernel sources, then
the patch can applied with the patch command. For instance:

patch -p1 < ~/uksm/v5.x/uksm-5.19.patch

If all goes well all hunks are applied. Just compile and install the kernel.

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
