# HorizonXI-ttimers
tTimers addon fix for hXI private server:

[pRealTime null pointer dereference]:
The addon uses a broken memory pattern scan (ashita.memory.find) to locate FFXI's real-time clock in FFXiMain.dll. hXI's Client is modified so that this scan can return 0 (not found). Code tries to dereference that null pointer inside CalculateBuffDuration(), which is called every time a buff packet arrives. Fix: 
-added a validity check on the pointer at scan time.
-wrapped memory reads in a pcall.
-bad reads fail instead of crashing.

[Unclamped duration values crashing D3DX]:
-When real-time pointer fails (or a corrupted packet comes in), buff durations can occasionally come back as NaN, negative infinity, or insanely large numbers.
-They get stored and then passed into D3DX sprite rendering math, which causes instant crash.
Fix: added sanity checks clamping any duration over 86400 seconds (24 hours) or below 0 to a safe value before being stored.

[GetAbilityByTimerId not existing on HorizonXI]
-hXI uses an older version of the Ashita framework that uses GetAbilityByTimerId differently
-The recast tracker calls it with no error handling, when absent: Lua throws a nil-call error and the addon crashes. 
Fix: wrapped in a pcall so if it's missing, the code falls back to linear ability table scan that already existed.

[D3DXCreateTextureFromFileInMemoryEx called @ -1 size]
-For status effect icons, the original code passed -1 as the SrcDataSize parameter to DX texture loader, relying on undefined behavior (some driver versions tolerate it, others don't). Implemented fix for hXI D3D9 proxy..
Fix: check for a valid ImageSize field on the status resource; if missing or zero, revert to a safe fixed size rather than passing -1.
