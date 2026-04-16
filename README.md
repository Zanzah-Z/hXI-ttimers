# HorizonXI-ttimers<br>
tTimers addon fix for hXI private server:<br><br>
[pRealTime null pointer dereference]:<br>
The addon uses a broken memory pattern scan (ashita.memory.find) to locate FFXI's real-time clock in FFXiMain.dll. hXI's Client is modified so that this scan can return 0 (not found). Code tries to dereference that null pointer inside CalculateBuffDuration(), which is called every time a buff packet arrives.<br>
Fix: <br>
-added a validity check on the pointer at scan time.<br>
-wrapped memory reads in a pcall.<br>
-bad reads fail instead of crashing.<br>
<br>
[Unclamped duration values crashing D3DX]:<br>
-When real-time pointer fails (or a corrupted packet comes in), buff durations can occasionally come back as NaN, negative infinity, or insanely large numbers.<br>
-They get stored and then passed into D3DX sprite rendering math, which causes instant crash.<br>
Fix: added sanity checks clamping any duration over 86400 seconds (24 hours) or below 0 to a safe value before being stored.<br>
<br>
[GetAbilityByTimerId not existing on HorizonXI]<br>
-hXI uses an older version of the Ashita framework that uses GetAbilityByTimerId differently<br>
-The recast tracker calls it with no error handling, when absent: Lua throws a nil-call error and the addon crashes. <br>
Fix: wrapped in a pcall so if it's missing, the code falls back to linear ability table scan that already existed.<br>
<br>
[D3DXCreateTextureFromFileInMemoryEx called @ -1 size]<br>
-For status effect icons, the original code passed -1 as the SrcDataSize parameter to DX texture loader, relying on undefined behavior (some driver versions tolerate it, others don't). Implemented fix for hXI D3D9 proxy.<br>
Fix: check for a valid ImageSize field on the status resource; if missing or zero, revert to a safe fixed size rather than passing -1.<br>
