# ReTerraForged Archipelago Terrain System

Island/archipelago terrain generation for **ReTerraForged**, a fork of [NeoTerraForged](https://github.com/equalizer32/NeoTerraForged/tree/1.20.1) for Minecraft 1.20.1 Forge.

This repository contains the source code and architecture documentation for an island terrain system that generates archipelagoes with irregular coastlines, continuous ocean-to-land height blending, multi-layer mountain terrain (ridges, hills, volcanoes), and proper Minecraft biome assignment.

## Features

- **Archipelago Generation** — Configurable island size, density, height, and shape using multi-layer warped simplex noise
- **Natural Coastlines** — Irregular, organic island shapes (not circular Voronoi islands)
- **Smooth Ocean-to-Land Transition** — Multi-segment hermite smoothstep height blending: ocean floor → shallow shelf → beach → inland
- **Mountain Terrain** — Three-layer noise (ridge + billow + volcano) with configurable mountain/volcano chance
- **Proper Beach Biomes** — ISLAND_BEACH terrain forced to Minecraft BEACH biome via multi-noise parameter synchronization
- **River Protection** — River carving smoothly fades on island mountains; river valley erosion override is skipped for island cells
- **Full UI Configuration** — 12 configurable parameters with real-time preview in the preset screen

## Upstream

This code is based on [NeoTerraForged 1.20.1](https://github.com/equalizer32/NeoTerraForged/tree/1.20.1) by equalizer32. See the original repository for the full TerraForged terrain generation framework.

## Contents

| Path | Description |
|------|-------------|
| `ARCHITECTURE.md` | Detailed architecture documentation in English |
| `src/` | Modified and new source files |
| `DEVELOPMENT_LOG.md` | Iteration history (Chinese) |

## License

MIT License — consistent with the upstream [TerraForged](https://github.com/dsmith/terraforged) project.

## Repository Scope

This repository is an architecture/reference extract, not a standalone buildable NeoTerraForged module. The files under `src/` show the important archipelago implementation paths and final terrain/biome logic, but they are not a complete replacement for the full mod source tree.

When porting this system, please check the surrounding integration points in the target version:

- `TerrainType.java`: island terrain registrations such as `ISLAND`, `ISLAND_BEACH`, and `ISLAND_MOUNTAINS`.
- `Preset.java` and `Presets.java`: preset serialization/default values for `IslandSettings`.
- `WorldLookup.java`: final `ISLAND_BEACH -> BEACH` remapping used by lookup/cache paths.
- `RTFTranslationKeys.java` and `RTFLanguageProvider.java`: UI labels/tooltips for the island settings page.
- River carving integration: `RiverTerrainFade.java` is included as the shared fade rule, but a full port also needs the corresponding `RiverCarver.java` and `Wetland.java` call-site changes.

Some uploaded files also reference the local fake-water-biome system (`FakeWaterBiomeResolver`, `FakeWaterBiomeTarget`, and `FakeWaterBiomeSettingsPage`). That system is separate from the archipelago terrain feature. For a clean 1.21.1 port, either port that system too or remove those references and keep only the archipelago-related logic.