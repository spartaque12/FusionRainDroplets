# Lens Drops for RTX Remix (GTA IV)

Fusion Rain Droplets as a [RemixFX](https://github.com/xoxor4d/dxvk-remix/blob/game/gta4/docs/RemixFx.md) effect, for GTA IV with xoxor4d's [RTX Remix compatibility mod](https://github.com/xoxor4d/gta4-rtx) (gta4-rtx). Under Remix the frame is path traced and put on the screen right before the HUD, so the plugin's own drops never show. This effect draws the same drops inside the runtime instead, below the HUD.

It is a port of this repository, not a remake:

- **Simulation:** `source/xrd/xrd.h` (`SpawnDrops`, `CalculateMovement`, `MoveDrop`, `NewTrace`, `Fade`), run on the GPU over a pool of up to 4096 drops.
- **Drawing:** the quads `AddToRenderList` builds, drawn with the Direct3D 9 pixel shader (`source/shaders/d3d9/drops.hlsl`), its light field (`light.hlsl`) and a mip chain of the frame.
- **Shapes:** the atlas of drop shapes (`lensdrops.png`, the same file as `source/resources/lensdrops.png`).

Two things differ from the plugin:

- **Light Steadiness:** the light a drop gathers is carried over from the frames before. Against the path traced frame, a small light such as a tail light makes the glow flicker otherwise. Set it to 0 for the plugin's behaviour.
- **Trails:** the bands of water are drawn as one body, by the distance to their middle line, rather than as quads, so they have no seams.

Snow is not ported.

## Requirements

- gta4-rtx with the lens values (xoxor4d/gta4-rtx#192). It writes what the game asks the lens for (rain, blood, splashes, the fountain spray, the game's frame time) as game values.
- A Remix runtime that binds effect parameters to game values (`param.<id>.gamevalue`, xoxor4d/dxvk-remix#3).
- Optional: the Slang compiler, only needed after editing the effect's source (see below). gta4-rtx releases don't include it.

## Shader compiler

The folder includes the compiled `lens_drops.remixfx.<entry>.spv` files (one per pass), so the effect loads without a compiler. A runtime that finds `slangc.exe` still rebuilds them when the source is newer.

The effect's source is Slang. With a compiler the runtime compiles it to SPIR-V when the game starts and writes `lens_drops.remixfx.*.spv` and `.spv.d` files beside it. Those are a cache: they are rebuilt whenever a `.slang` or `.slangh` file is newer, and deleting them forces a rebuild. Without a compiler and without those files the effect does not load.

1. Get the Windows x64 build of Slang from https://github.com/shader-slang/slang/releases (tested with 2025.10.4).
2. Copy `slangc.exe` and its `slang*.dll` files from the archive's `bin` folder into a `slang` folder next to `GTAIV.exe` (`slang/slangc.exe`). `slang/` next to the runtime (`.trex/slang/`) works too. `slang-llvm.dll` (about 72 MB) is not needed for this effect, which leaves about 28 MB.
3. Start the game. The first start compiles the effect, which takes a few seconds; later starts reuse the cache.

If the effect is missing, `rtx-remix/logs/remix-dxvk.log` shows the compiler's messages.

## Installation

Copy this folder into `remix-shaders` next to `GTAIV.exe` (`remix-shaders/lens_drops/`). The effect shows under External Effect Files in the Remix menu (Alt+X), where its settings are: the ones of the plugin's `.cfg`, plus Light Steadiness. Force Rain shows drops in any weather.

The plugin can stay installed. When gta4-rtx is loaded it stays out of the way, since both hook the same camera effect requests of the game.

`remixfx_bindings.slangh` is the runtime's RemixFX ABI header (MIT), shipped beside the effect as the runtime intends.
