<!--
SPDX-FileCopyrightText: 2026 shadPS4 Emulator Project
SPDX-License-Identifier: GPL-2.0-or-later
-->

# Fix shader issue for game Love Live! School Idol Festival after school ACTIVITY Wai-Wai!Home Meeting!! (CUSA24620)

It solves the dark character shading problem mentioned in [this](https://github.com/shadps4-compatibility/shadps4-game-compatibility/issues/2664)

Before:
<img width="1919" height="1068" alt="image" src="https://github.com/user-attachments/assets/b860d226-7633-4ae6-afef-9a7834154a3d" />

After:
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/4afe2309-401a-4eb5-aa22-91ead30291f1" />


TLDR apparently caused by 

```cpp
bool Rasterizer::IsComputeImageClear(const Pipeline* pipeline) {
    if (!pipeline->IsCompute()) {
        return false;
    }

```
inside src/video_core/renderer_vulkan/vk_rasterizer.cpp

All I did (or the AI told me to) was:
```cpp
bool Rasterizer::IsComputeImageClear(const Pipeline* pipeline) {
    return false;
    if (!pipeline->IsCompute()) {
        return false;
    }
```

### I am not 100% sure if this change would cause other problem in other games other than CUSA24620 so use this at your own risk!

here was what the AI told me if anyone wants an explaination:

> The bug is not in the shaders themselves; it is a false positive in the emulator's compute image clear heuristic (IsComputeImageCopy).The Exact Bug MechanismIn newer versions (starting with PR #3795 / commit 555be14, "vk_rasterizer: Optimize full image compute clears"), shadps4 tries to optimize performance by detecting compute shaders that wipe or memset memory.When SIFAC runs, it executes compute shader cs_0x000000007eb5062f to generate a dynamic $120 \times 67$ lighting mask.Because this compute shader only reads the first dword (out of a 16-byte view), shadps4's new heuristic erroneously flags it as a memset/clear.The emulator wipes the $120 \times 67$ buffer to pure zeroes.When the game's untile pass attempts to calculate character skin and costumes, the red channel of the lighting mask evaluates to $0.0$, multiplying the character's albedo by 0 and rendering everything pitch black. Instead of porting an entire outdated file, you only need to prevent the clear heuristic from running on that specific lighting pass.

