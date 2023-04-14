# uksm
The uksm were orginally maintained in [this](https://github.com/dolohow/uksm) repository.
However it looks like the most recent patch has been for 5.17.

# Why?
I'm currently working at Meta on improving KSM. I'm using the UKSM implementation to
compare it against KSM. I needed a more current implementation of UKSM to compare it
to KSM. I ported this UKSM patch to more current releases. If you use the patches,
please add some reference to this repository.

# Versions
| Version | Changes                           | Testing                         |
|---------|-----------------------------------|---------------------------------|
| 5.18    | Some minor changes for folios     | Basic testing                   |
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
