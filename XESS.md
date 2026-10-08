# Intel XeSS for Godot

This fork integrates Intel XeSS Super Resolution as a native 3D scaling mode,
alongside the upstream bilinear and FSR modes.

- **Platform:** Windows only.
- **Renderer:** Forward+ only.
- **Graphics API:** Vulkan or Direct3D 12.

## Getting the runtime

The release archive already contains `libxess.dll`, so nothing extra is needed to
run the prebuilt editor.

Only the public XeSS headers are vendored (`thirdparty/intel-xess`, SDK v3.0.1);
the runtime itself is not in this repository. If you build from source, supply it:

1. Download a release from <https://github.com/intel/xess/releases> (v3.0.1 or
   newer).
2. Copy `bin/libxess.dll` (the **x64** build) next to the Godot executable.

The library is loaded lazily with `LoadLibrary`, so a missing DLL is not fatal —
XeSS simply reports itself unavailable and the renderer falls back to FSR 2.

> The SDK package version and the XeSS-SR library version are different numbers.
> SDK 3.0.2 ships a `libxess.dll` that reports `2.0.2` from `xessGetVersion`, which
> is expected.

## Enabling it

Project-wide, in **Project Settings → Rendering → Scaling 3D**:

- `rendering/scaling_3d/mode` → `Intel XeSS (Slow)`
- `rendering/scaling_3d/scale` → the render scale, e.g. `0.5` for 2x upscaling.
  `1.0` runs XeSS at native resolution as an anti-aliasing pass.

Per viewport, set `Viewport.scaling_3d_mode` to `SCALING_3D_MODE_XESS`, or from
the server `RenderingServer.VIEWPORT_SCALING_3D_MODE_XESS`.

The editor's own 3D viewport follows the project setting, so changing it there is
enough to preview the result.

## Fallback behaviour

XeSS steps aside instead of failing:

| Situation | Result |
| --- | --- |
| Built without XeSS (`use_xess=no`, or not Windows) | FSR 2, warns once |
| `libxess.dll` missing, or context creation fails | FSR 2 |
| Mobile or Compatibility renderer | Bilinear, warns once |
| Set on a non-Forward+ renderer through the API | Rejected, warns once |

## Verifying

Run with `--verbose` and look for:

```
Intel XeSS runtime loaded (version 3.0.1).
Intel XeSS: creating a Vulkan context for 576x324 -> 1152x648.
```

The second line names the active backend (`Vulkan` or `Direct3D 12`) and the
resolutions in use. If it is missing, XeSS never activated and you are looking at
the FSR 2 fallback.

## Building

`use_xess` is on by default and needs `vulkan=yes` or `d3d12=yes`:

```
scons platform=windows target=editor vulkan=yes d3d12=yes
scons platform=windows target=editor use_xess=no    # opt out
```

Only the vendored headers are needed to build; nothing links against the XeSS
import library.

## Exporting a project

An exported game runs on an export template, not on the editor binary, so the
templates have to carry XeSS too.

### Installing the templates

Download `Godot_v4.7.2-stable-xess_export_templates.tpz` from the release and
install it with **Editor → Manage Export Templates → Install from File**. This
fork reports its version as `4.7.2.stable.xess`, so the templates land in their
own slot and leave an existing official `4.7.2.stable` install untouched.

### Building them yourself

```
scons platform=windows target=template_release vulkan=yes d3d12=yes
scons platform=windows target=template_debug vulkan=yes d3d12=yes
```

Register the results as **Custom Template → Release / Debug** in the export
preset, or install them as a template version.

The exported executable needs `libxess.dll` beside it, and the export copies it
for you: put the runtime next to the export templates, named for the target
architecture.

```
libxess.x86_64.dll    next to the template binaries
```

It is copied into the export folder as `libxess.dll`. This mirrors how the engine
already ships the ANGLE, Direct3D 12 Agility SDK and PIX runtimes.

**Export → Options → Application → Export Xess** controls it:

| Value | Behaviour |
| --- | --- |
| `Auto` (default) | Copy when the project uses XeSS on the Forward+ renderer |
| `Yes` | Always copy, and warn if the runtime is not next to the templates |
| `No` | Never copy |

Shipping the runtime is the developer's responsibility; check the Intel XeSS SDK
license for the redistribution terms that apply to your game.

## About this fork

This fork was written with the help of a generative AI assistant, and verified by
building and running it on real hardware.

이 포크는 생성형 AI의 도움을 받아 작성했고, 실제 하드웨어에서 빌드하고 동작을
확인했습니다.
