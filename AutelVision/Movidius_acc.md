## 代码优化加速
1. Avoid direct DDR access where possible. If required, ensure L1 and L2
caches are enabled.
2. Access data via CMX where possible
3. Do not access the same CMX tile (see boxes in Section 4.4.3) from both
load store ports in the same cycle.
4. Ensure that code and frequently used data do not both reside in the same
CMX tile. Try putting code into DDR and using L1 instruction cache for that.
5. Avoid vector accesses to unaligned addresses
6. Slow access if use uncatched view variables.

SHAVE访问CMX的时候，一直是uncatched access.
