# Windows Lunar Lake integration

The provider wheel accepts `--xpu-target lnl` for the experimental Windows
Lunar Lake profile. The target is metadata used to keep the provider manifest
and the native `omni_xpu_kernel` AOT identity aligned; it does not claim that
all allocator or ComfyUI workflows are validated on Windows LNL.

The provider remains opt-in through the existing DynamicVRAM integration.
Record a target-local wheel hash and workflow receipt before treating the
profile as validated.
