# AIMDO XPU runtime provider wheel

This builder converts an already-built XPU `comfy-aimdo` wheel into the
co-installable `comfy-aimdo-xpu-runtime` provider distribution. The provider
owns only `comfy_aimdo_xpu_runtime`; Python modules and the Level Zero native
library are stored below its private `_vendor` directory. It never installs a
top-level `comfy_aimdo` file, so the official AIMDO distribution remains
independently upgradeable.

After building `comfy_aimdo/aimdo_xpu.so` and the canonical wheel, run:

```bash
python packaging/xpu_runtime_provider/build_wheel.py \
  --source-wheel dist/comfy_aimdo-0.4.15-cp39-abi3-linux_x86_64.whl \
  --output-dir dist/provider \
  --source-revision "$(git rev-parse HEAD)" \
  --torch-version 2.13.0+xpu \
  --xpu-target bmg
```

The output contains a lightweight
`comfyui_omnixpu.runtime_providers` entry point and a manifest covering the
canonical version, exact source revision, source-wheel hash, native-library
hash, supported runtime, and allocator modes. Importing its metadata does not
import PyTorch or AIMDO.

When the source contains `malloc_graph.py`, the builder requires the complete
provider module set and all twelve compiler ABI/provenance exports in the native
library. This source currently declares the finite official API combinations
`0.4.15` and `0.5.2`, with upstream semantic reference
`9a688a905b85402ef696beedaab8313a9759cc52`. The provider distribution keeps the
source wheel's existing version. Source revision and native content hashes
identify development changes independently of that version.

`control.get_memory_compiler_capability()` reports the built core separately
from runtime availability. XPU recording remains unavailable until a logical
allocation router and its lifetime contract are implemented and validated;
explicit `record(xpu_stream)` raises an unsupported error. `malloc_graph` and
`control` must resolve to the same provider directory. A missing local module or
native ABI prevents initialization before allocator installation.

ComfyUI-OmniXPU activates this provider only when DynamicVRAM is explicitly
enabled and the official AIMDO attempt has left no live native or allocator
state. Linux selects the global XPU pluggable allocator; Windows selects the
native Unified Runtime hook. A failure after either becomes live is fatal
because allocator ownership cannot be rolled back safely.

Run the portable provider and Linux source-contract tests inside the target
development container:

```bash
python -m pytest -q \
  tests/test_xpu_runtime_provider_wheel.py \
  tests/test_linux_xpu_source_contract.py
```
