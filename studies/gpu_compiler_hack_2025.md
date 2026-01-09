# How to add a lowering pass to the MESA NIR compiler

## Introduction

This document shows how to **add, register, and test a custom lowering pass** in the **Mesa NIR GPU compiler**.
The pass rewrites fragment shader outputs at the NIR level, allowing framebuffer manipulation and observation.

> ⚠️ Note  
> Initial drafts were GPT-assisted, but **final logic, debugging, integration, and validation were done manually**.

---

## Environment & Requirements

### System
- **OS**: Fedora Linux 43 (Workstation)
- **Kernel**: 6.17 (x86_64)
- **GPU Target**: AMD Radeon (x86 only)
- **Compositor**: Wayland (X11 additionally enabled)

### Mesa
- **Mesa version**: `26.0.0-devel`
- **Git commit**: `>= 78e1f53429`

### Build Tools
- **Build system**: Meson 1.8.5
- **Builder**: Ninja 1.13.1
- **Compiler**: GCC
- **Python**: 3.14.2

### Mesa Build Configuration

```bash
meson setup build \
  --prefix=/usr/local \
  --buildtype=debug \
  -Dplatforms=wayland,x11 \
  -Dgallium-drivers=radeonsi \
  -Dvulkan-drivers=amd \
  -Dllvm=enabled \
  -Dshared-glapi=enabled \
  -Dglx=dri \
  -Degl=enabled \
  -Dgbm=enabled \
  -Dgles1=disabled \
  -Dgles2=enabled
```

---

## 1. Implement a Custom NIR Pass

**Location**
```
src/compiler/nir/
```

**Files added**
- `nir_tint_pass.h`
- `nir_tint_pass.c`

### Header

```c
#ifndef TINT_PASS_H
#define TINT_PASS_H

#include "nir.h"

bool nir_tint_fragment_outputs(nir_shader *shader);

#endif
```

### Implementation

```c
#include "nir.h"
#include "nir_builder.h"
#include "compiler/nir/nir_tint_pass.h"

bool nir_tint_fragment_outputs(nir_shader *shader)
{
    /* Only fragment shaders */
    if (shader->info.stage != MESA_SHADER_FRAGMENT ||
        shader->info.separate_shader)
        return false;

    nir_function_impl *impl = nir_shader_get_entrypoint(shader);
    nir_builder b = nir_builder_create(impl);

    bool progress = false;

    nir_foreach_block(block, impl) {
        nir_foreach_instr_safe(instr, block) {

            if (instr->type != nir_instr_type_intrinsic)
                continue;

            nir_intrinsic_instr *intr =
                nir_instr_as_intrinsic(instr);

            /* Target framebuffer output stores */
            if (intr->intrinsic != nir_intrinsic_store_output)
                continue;

            nir_io_semantics sem =
                nir_intrinsic_io_semantics(intr);

            if (sem.location != FRAG_RESULT_COLOR &&
                sem.location != FRAG_RESULT_DATA0)
                continue;

            /* Inject before store */
            b.cursor = nir_before_instr(&intr->instr);

            /* Force solid red output */
            nir_def *v =
                nir_imm_vec4(&b, 1.0f, 0.0f, 0.0f, 1.0f);

            nir_src_rewrite(&intr->src[0], v);
            progress = true;
        }
    }

    /* Required: invalidate metadata */
    impl->valid_metadata = 0;
    return progress;
}
```

---

## 2. Register the Pass in Meson

**File**
```
src/compiler/nir/meson.build
```

Add the source file so it is compiled and linked into libnir:

```meson
files_libnir += files(
  'nir.c',
  'nir.h',
  'nir_tint_pass.c',  # custom pass
  ...
)
```

---

## 3. Hook the Pass into RadeonSI (Call Site)

**File**
```
src/gallium/drivers/radeonsi/si_shader_nir.c
```

### Environment Variable Gate

Use an environment variable to enable or disable the pass without rebuilding:

```c
static bool tint_enabled_cached = false;
static bool tint_checked = false;

static inline bool tint_enabled(void)
{
    if (!tint_checked) {
        const char *e = getenv("MESA_TINT");
        tint_enabled_cached =
            (e && e[0] && strcmp(e, "0") != 0);
        tint_checked = true;
    }
    return tint_enabled_cached;
}
```

### Injecting the Pass

The custom pass is inserted **after fragment IO lowering** and **before final metadata requirements**:

```c
void si_finalize_nir(struct pipe_screen *screen,
                     struct nir_shader *nir,
                     bool optimize)
{
    if (!nir->info.io_lowered) {
        nir_lower_io_passes(nir, false);
        NIR_PASS(_, nir,
                 nir_remove_dead_variables,
                 nir_var_shader_in |
                 nir_var_shader_out, NULL);
    }

    if (nir->info.stage == MESA_SHADER_FRAGMENT) {
        NIR_PASS(_, nir,
                 si_nir_lower_color_inputs_to_sysvals);
        NIR_PASS(_, nir,
                 nir_recompute_io_bases,
                 nir_var_shader_out);
        if (tint_enabled())
            NIR_PASS(_, nir,
                     nir_tint_fragment_outputs);
    }
    
     /* ...etc codes... */
     
    /* Later passes require divergence metadata */
    nir_metadata_require(
        nir_shader_get_entrypoint(nir),
        nir_metadata_divergence);
}
```

> Because `nir_metadata_divergence` is required later,
> **custom passes must invalidate metadata explicitly**.

---

## 4. Build and Install Mesa

```bash
ninja -C build
sudo ninja -C build install
```

Installed libraries will reside under:
- `/usr/local/lib`
- `/usr/local/lib64`

---

## 5. Test Program: Framebuffer Hash Verification

This program:
1. Creates an offscreen EGL context
2. Renders a fragment shader
3. Reads back the RGBA framebuffer
4. Hashes the result with SHA-256

It is used to **verify that the NIR pass overrides fragment output correctly**.

### Build

```bash
gcc -g -O0 dumpgl.c -o dumpgl \
  -L/usr/local/lib64 \
  -Wl,--enable-new-dtags \
  -Wl,-rpath,/usr/local/lib64 \
  -lEGL -lGL -lcrypto -lm
```

### Run

```bash
MESA_TINT=1 ./dumpgl
```

If the pass is active, the framebuffer hash should correspond to a
solid red image regardless of shader logic.

---

## References

- Mesa NIR Documentation  
  https://docs.mesa3d.org/nir/index.html
