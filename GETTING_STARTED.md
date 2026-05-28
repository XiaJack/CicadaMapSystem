# CicadaMapSystem - Getting Started Guide

## Quick Start

### 1. Clone and Build

```bash
git clone https://github.com/XiaJack/CicadaMapSystem.git
cd CicadaMapSystem
mkdir build && cd build
cmake ..
cmake --build . --config Release
```

### 2. Run Tests

```bash
ctest
# or
./cicada_tests
```

### 3. Include in Your Project

```cpp
#include "src/core/Map.h"
#include "src/editor/MapEditor.h"
#include "src/utils/Performance.h"

using namespace Cicada;
```

## Core Concepts

### Map
The main container that holds all layers, manages chunks, and provides tile access.

```cpp
MapConfig config;
config.type = MapType::GRID;
config.width = 1024;
config.height = 1024;
config.infinite = true;
config.num_layers = 3;

Map map(config);
map.initialize();
```

### Layers
Independent 2D tile grids that render on top of each other. Each layer can have unique properties.

```cpp
LayerId terrain = map.addLayer(LayerType::TERRAIN, "Terrain", 0);
LayerId objects = map.addLayer(LayerType::OBJECT, "Objects", 1);
```

### Tiles
Individual cells in a layer. Each tile has an ID and additional properties.

```cpp
map.setTile(terrain, {10, 20}, 42);  // Set tile ID 42 at position (10, 20)

if (auto tile = map.getTile(terrain, {10, 20})) {
    tile->setRotation(2);
    tile->setFlag(COLLISION_FLAG);
}
```

### Chunks
Fixed-size sections of the map (32x32 tiles by default). Chunks are automatically loaded/unloaded based on viewport.

```cpp
auto visible_chunks = map.getVisibleChunks();
for (const auto& chunk : visible_chunks) {
    // Render chunk
}
```

### Viewport
Defines the camera view used for culling visible chunks.

```cpp
Viewport viewport;
viewport.position = {0.0f, 0.0f};
viewport.size = {1280.0f, 720.0f};
viewport.zoom = 1.0f;

map.setViewport(viewport);
```

## Common Tasks

### Fill an Area with Tiles

```cpp
// Fill rectangle from (0,0) to (100,100) with tile ID 5
map.fillRect(terrain_layer, {0, 0}, {100, 100}, 5);
```

### Clear an Area

```cpp
// Clear rectangle from (50,50) to (150,150)
map.clearRect(terrain_layer, {50, 50}, {150, 150});
```

### Create a Terrain Brush for Editing

```cpp
TerrainBrush brush;
brush.setSize(5);
brush.setShape(TerrainBrush::BrushShape::CIRCLE);
brush.setMode(TerrainBrush::BrushMode::PAINT);
brush.setTileId(42);
brush.setLayer(terrain_layer);

// Apply brush at world position
brush.apply(map, {256, 256});
```

### Implement Undo/Redo

```cpp
MapEditor editor(map);
editor.setBrush(brush);

// Make edits
editor.applyBrush({100, 100});
editor.applyBrush({150, 150});

// Undo last edit
if (editor.canUndo()) {
    editor.undo();
}

// Redo
if (editor.canRedo()) {
    editor.redo();
}
```

### Monitor Performance

```cpp
auto& perf = Performance::getInstance();

// In game loop
perf.beginFrame();
// ... game logic ...
perf.endFrame();

// Get stats
printf("FPS: %u\n", perf.getFPS());
printf("Frame Time: %.2f ms\n", perf.getFrameTime() * 1000.0f);
```

## API Reference

### Map Class

| Method | Description |
|--------|-------------|
| `initialize()` | Initialize map and layers |
| `shutdown()` | Cleanup resources |
| `addLayer()` | Add new layer |
| `getLayer()` | Get layer by ID |
| `setTile()` | Set tile at position |
| `getTile()` | Get tile at position |
| `fillRect()` | Fill rectangle with tiles |
| `clearRect()` | Clear rectangle |
| `setViewport()` | Update camera viewport |
| `getVisibleChunks()` | Get chunks in viewport |
| `getMemoryUsage()` | Get total memory usage |

### Layer Class

| Method | Description |
|--------|-------------|
| `getTile()` | Get tile at local position |
| `setTile()` | Set tile at local position |
| `clearTile()` | Clear tile at position |
| `resize()` | Resize layer |
| `setOpacity()` | Set layer opacity |
| `setVisible()` | Set layer visibility |
| `setParallax()` | Set parallax effect |

### MapEditor Class

| Method | Description |
|--------|-------------|
| `paintTile()` | Paint single tile |
| `eraseTile()` | Erase single tile |
| `fillRect()` | Fill rectangle |
| `applyBrush()` | Apply brush at position |
| `undo()` | Undo last edit |
| `redo()` | Redo last undo |
| `canUndo()` | Check if undo available |
| `canRedo()` | Check if redo available |

## Configuration

### Chunk Manager

Adjust memory usage and performance:

```cpp
// Maximum 512 chunks in memory
chunk_manager_->setMaxChunks(512);
```

### Performance Monitoring

Enable performance tracking:

```cpp
auto& perf = Performance::getInstance();
perf.reset();  // Clear statistics
```

## Troubleshooting

### Out of Memory
- Reduce `MAX_CHUNK_POOL` if too many chunks are loaded
- Monitor `map.getMemoryUsage()`
- Check viewport size and zoom level

### Low Frame Rate
- Reduce number of visible layers
- Increase chunk size (if possible)
- Profile with `Performance` class
- Reduce viewport size

### Coordinate Issues
- Ensure coordinate system type matches map type
- Use `worldToChunkCoord()` for chunk lookups
- Use `getTileAtScreen()` for mouse picking

## Examples

See the `example/` directory for:
- BasicMapUsage.cpp - Complete map setup
- CoordinateSystemExample.cpp - Coordinate conversion
- MapEditingExample.cpp - Editing with undo/redo
- PerformanceProfilingExample.cpp - Performance monitoring

## Next Steps

1. Read the [Architecture Overview](ARCHITECTURE.md)
2. Check out the [API Documentation](API.md)
3. Review example code in `example/`
4. Run the test suite: `ctest`
5. Integrate into your game engine

## Support

For issues, questions, or improvements:
- Create a GitHub Issue
- Check existing issues for solutions
- Review test cases for usage examples
