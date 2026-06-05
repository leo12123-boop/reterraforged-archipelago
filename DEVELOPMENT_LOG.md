# ReTerraForged 模组开发 - 任务计划

## 总体目标

为 Minecraft 1.20.1 Forge 的 ReTerraForged 地形模组添加：
1. ✅ **山脉群系区域覆盖**（已完成，测试通过）
2. ✅ **群岛地形系统**（v14 已完成，沙滩群系已修复并通过测试）
3. ✅ **河流断流平滑修复**（已完成，编译通过）

---

## 阶段 1：山脉群系优化 [complete] [verified]

让山脉自成一个完整群系，且群系与周围温度带匹配。

### v1 - 独立山脉噪声 [失败]
- 添加 `mountainTemperature`/`mountainMoisture` 独立噪声
- HIGHLAND + LOWLAND-hills 覆盖
- 失败原因：采样点在 biome 中心而非 terrain 中心，山脉仍多群系

### v2 - 恢复 Climate.java [失败]
- 移除 LOWLAND-hills 覆盖，恢复原始逻辑
- 失败原因：未触及根因

### v3 - 地形区域中心 [失败]
- Cell.java 新增 `terrainRegionCenterX/Z`
- RegionModule.java 存储地形区域中心
- 失败原因：群系不匹配气候带

### v4 - 改用已有气候噪声 [buggy]
- 删除独立噪声，改用 `this.temperature`/`this.moisture`
- Bug：世界坐标未转频率空间 → 沙漠山脉出雪原

### v5 - 修复坐标缩放 [成功]
- `terrainRegionCenterX * this.biomeFreq` 坐标缩放
- 补充 `cell.temperature`/`cell.moisture` 同步
- 测试结果：圆满成功

### 山脉群系最终改动文件
| 文件 | 改动 |
|---|---|
| `Cell.java` | +`terrainRegionCenterX/Z` 字段 |
| `RegionModule.java` | Voronoi 后存储地形区域中心 |
| `ClimateModule.java` | HIGHLAND 覆盖 + 频率缩放 |

---

## 阶段 2：群岛地形系统 [complete]

在海洋中生成群岛，要求：
- 不规则海岸线、沙滩过渡
- 山脉/火山地形
- UI 可调参数
- 真实的岛屿→海洋过渡（无断崖）

### v1 - 初始实现 [失败]
- 添加 3 种岛屿地形类型 + IslandSettings + UI
- 使用 simplex 噪声生成地形
- 失败：扁平地形、河流群系、垂直断崖

### v2 - 复用大陆管线 [失败]
- 改为修改 `continentEdge` 让大陆管线处理
- 失败：参数失效、椭圆形状、仍然断崖

### v3 - 自定义噪声 + smoothstep [失败]
- 恢复自定义噪声，smoothstep 高度过渡
- 失败：岛屿→海洋笔直断崖 + 椭圆形状

### v4 - 完全复用大陆管线 [失败]
- 删除 ArchipelagoPopulator，改用 ArchipelagoPlacer
- 失败：参数全失效、Still 断崖

### v5 - 自定义噪声 + 海底梯度混合 [失败]
- 恢复自定义噪声，blendAlpha = elevation * 2
- 失败：岛屿聚类不均匀、断崖

### v6 - 连续影响值 + Worley 密度 [失败]
- 废除二进制阈值，连续影响值混合
- Worley 噪声均匀分布
- 失败：三角形图案、密度调节失效

### v7 - 两级高度混合 + 海岸参数 + 山脉保护 [失败]
- 两级高度：浅海架→岛屿
- 新增 offshoreDepth/beachWidth/beachCoverage
- 失败：浅海架目标低于海底，过渡无效果

### v8 - 修复浅海架 + 山脉中心约束 [失败]
- 浅海架目标固定在海面下
- 山脉限制在 islandAlpha > 0.7
- 测试：无任何改善

### v9 - 连续岛屿 alpha + 分段海岸过渡 + 山脉边缘衰减 [compile-verified]
- `sizeNoise`/`densityNoise` 改为连续 `islandAlpha`，密度只在阈值附近淡入
- 移除强制 `levels.water + 0.01F` 出水
- 高度分段为海底→浅海架→沙滩/地面→岛内目标高度
- 山脉高度和 `ISLAND_MOUNTAINS` 类型切换统一受 `mountainAlpha` 控制
- `IslandBlender` 允许 `COAST` 参与群岛过渡
- 验证：`:common:compileJava` 成功；地形/山脉已修复

### v10 - 群系参数 + 山脉重构 + 参数直觉化 [compile-verified]
- `cell.continentEdge` 分段写入 deepOcean→shallowOcean→coast→inland
- 山脉改为 ridge + billow + volcano 三层噪声 + mountainChance/volcanoChance
- `islandHeight` 直接参与 baseHeight + reliefHeight
- `CellSampler` 新增 `ISLAND_BEACH` → COAST 分支
- `ClimateModule` 岛屿群系覆盖陆地温湿度 + `modifyTerrain` guard
- `Heightmap.applyClimate()` riverMask guard 保护岛屿地形

### v10-fix - 沙滩群系命中尝试（4次迭代未完全解决）
- 沙滩区 `continentEdge` 拉宽到 `[shallowOcean, coast]`
- 沙滩侵蚀改为 `LEVEL_2` + `weirdness=0`
- 测试：UI 预览 ISLAND_BEACH 正确，地形正常，游戏中仍显示"河流"群系
- 根因：Minecraft 多噪声选择器参数组合未命中 beach

### v11 - 沙滩覆盖率与 river 命中参数修复 [compile-verified]
- `beachEnd` 从 `0.65F` 上限提高到 `0.85F`，并改为 `0.5 + beachCoverage * 1.5` 线性权重
- `mountainStart` 上限从 `0.65F` 提高到 `0.9F`，避免宽沙滩配置下山体过早贴边
- `ISLAND_BEACH` 的 `weirdness` 从 `0` 改为 `LOW_SLICE_NORMAL_DESCENDING`，避开 vanilla river 常用的 VALLEY 段
- `CellSampler` 中 `ISLAND_BEACH` continentalness 改为 `COAST.mid()` → `COAST.max()`，推向 coast 靠内侧

### v12 - 沙滩参数修复第二步 [test: forest]
- beachErosion 改为 LEVEL_4 + MID_SLICE_NORMAL_DESCENDING
- CellSampler ISLAND_BEACH 恢复 COAST 全段
- ClimateModule ISLAND_BEACH 固定 GRASSLAND + LEVEL_2 temp/moist
- 测试结果：岛屿边缘变成森林群系（COAST + LEVEL_4 + MID weirdness → 森林）

### v13 - CellSampler 统一固定沙滩多噪声参数 [compile-verified]
- 根因：前 3 轮调参均无法稳定命中原版沙滩超立方体，因为 5 个参数由独立字段分时计算，微小偏移就会落入相邻群系
- 方案：利用 NoiseRouter 评估顺序（CONTINENT → EROSION → WEIRDNESS），在 `CONTINENT.read()` 的 `ISLAND_BEACH` 分支中统一设置参数
- ClimateModule：ISLAND_BEACH 固定 `SAVANNA + LEVEL_3 temp + LEVEL_1 moisture`

### v14/final - ISLAND_BEACH -> BEACH 地形重映射 + LEVEL_4 侵蚀 [confirmed working]
- 关键洞察：CellSampler 参数竞态导致无法稳定命中 BEACH 群系
- 两层修复方案：
  1. `Heightmap.apply()` 将 `ISLAND_BEACH -> TerrainType.BEACH` 立即重映射，地形层变为标准 BEACH
  2. `CellSampler.CONTINENT` 读取 ISLAND_BEACH 并返回 COAST 范围大陆度
  3. `CellSampler.EROSION/WEIRDNESS/TEMPERATURE/MOISTURE` 全部返回固定值 (LEVEL_4, MID_SLICE_NORMAL_DESCENDING, LEVEL_3, LEVEL_1)
  4. `CellSampler.Cache2d.getAndUpdate()` 同步写入所有 4 个参数
- 结果：ISLAND_BEACH 区域现在可靠地渲染为 BEACH 群系

---

## 阶段 3：河流断流平滑 [complete] [compile-verified] [game-test-pending]

修复河流在高海拔或山脉地形断流时偶发无平滑断崖的问题。

### v1 - 统一断流 fade [compile-verified]
- 新增 `rivermap/fade/RiverTerrainFade.java` 独立分区，集中计算高度断流、山脉断流和河流标记阈值
- `RiverCarver.java` 移除高于 `fadeEndHeight` 的硬 `return`，改为 `heightFade` 自然衰减到 0
- `RiverCarver.java` 将河谷、河岸、河床拆成 `valleyFade` / `banksFade` / `bedFade` 三层衰减
- 山脉判断从硬编码 `TerrainType.MOUNTAINS_* / VOLCANO*` 改为 `cell.terrain.isMountain()`，自动覆盖 `ISLAND_MOUNTAINS` 和未来新增山脉
- 山脉区域保留少量河谷收口，河岸/河床逐渐抑制，避免山体被水渠切穿
- `Wetland.java` 复用同一套 fade，移除重复山脉 ID 判断和高海拔硬断流

---

## 已解决难点

| 问题 | 尝试次数 | 状态 |
|---|---|---|
| 岛屿→海洋笔直断崖 | 9 次 | ✅ v9 已修复 |
| 山脉在岛屿边缘产生切口 | 9 次 | ✅ v9 已修复 |
| 密度/大小参数解耦 | 4 次 | ✅ v10 保留参数语义 |
| 参数控制保留 | 5 次 | ✅ v10 通过编译 |
| 不规则海岸线 | 3 次 | ✅ v9 保留 warped simplex 外形 |
| 沙滩群系命中 | 7 次 | ✅ v14 最终修复，强制为 BEACH |

## 已知的成功实现

| 功能 | 状态 | 说明 |
|---|---|---|
| 山脉群系区域覆盖 | ✅ 完成 | 见阶段 1 v5 |
| 群岛 UI 配置页面 | ✅ 完成 | 12 个参数，运行正常 |
| 群岛地形类型注册 | ✅ 完成 | ISLAND/BEACH/MOUNTAINS |
| 岛屿基本生成 | ✅ 完成 | 地形/高度/山脉形态正常 |
| ISLAND_BEACH terrain 层 | ✅ 完成 | UI 预览正确 |
| ISLAND_BEACH continentalness | ✅ 完成 | COAST [-0.19,-0.11] |
| 岛屿 climate/biome 覆盖 | ✅ 完成 | 不再显示 ocean/frozen_ocean |
| CellSampler 统一固定沙滩参数 | ✅ v14 完成 | EROSION LEVEL_4 + Weirdness MID_SLICE_NORMAL_DESCENDING |
| 最终 Minecraft 沙滩群系 | ✅ 完成 | Heightmap ISLAND_BEACH -> BEACH 重映射 + Cache2d 同步参数写入 |