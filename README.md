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