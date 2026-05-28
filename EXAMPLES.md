# CicadaMapSystem Examples

Complete working examples for all major features.

## Example 1: BasicMapUsage.cpp
**Location**: `example/BasicMapUsage.cpp`

Demonstrates:
- Map creation and initialization
- Layer management
- Tile operations
- Viewport management
- Game loop integration
- Performance monitoring

**Run**: 
```bash
# After building
./basic_map_usage_example
```

## Example 2: CoordinateSystemExample.cpp
**Location**: `example/CoordinateSystemExample.cpp`

Demonstrates:
- Grid coordinate conversions
- Isometric coordinate projections
- Hexagonal coordinate systems
- World to screen conversions
- Screen to world conversions
- Tile picking

**Run**:
```bash
./coordinate_system_example
```

## Example 3: MapEditingExample.cpp
**Location**: `example/MapEditingExample.cpp`

Demonstrates:
- Map editor initialization
- Terrain brush setup
- Brush application
- Undo/redo operations
- Rectangle fill and clear
- Edit action stacks

**Run**:
```bash
./map_editing_example
```

## Example 4: PerformanceProfilingExample.cpp
**Location**: `example/PerformanceProfilingExample.cpp`

Demonstrates:
- Performance monitoring
- FPS calculation
- Frame time measurement
- Memory usage tracking
- Statistics collection
- Large map simulation

**Run**:
```bash
./performance_profiling_example
```

## Code Snippets

### Creating a 2D Game World

```cpp
#include "src/core/Map.h"
#include "src/editor/MapEditor.h"

using namespace Cicada;

int main() {
    // Configuration
    MapConfig config;
    config.type = MapType::GRID;
    config.width = 512;
    config.height = 512;
    config.infinite = true;
    config.num_layers = 3;
    
    // Create map
    Map map(config);
    map.initialize();
    
    // Add layers
    LayerId terrain = map.addLayer(LayerType::TERRAIN, "Terrain", 0);
    LayerId objects = map.addLayer(LayerType::OBJECT, "Objects", 1);
    LayerId effects = map.addLayer(LayerType::DECORATION, "Effects", 2);
    
    // Fill with terrain
    map.fillRect(terrain, {0, 0}, {511, 511}, 1);  // Grass
    
    // Place some objects
    map.setTile(objects, {100, 100}, 50);  // Tree
    map.setTile(objects, {200, 200}, 51);  // Rock
    
    // Cleanup
    map.shutdown();
    return 0;
}
```

### Implementing a Camera System

```cpp
class Camera {
    Cicada::Viewport viewport;
    Cicada::Map& map;
    
public:
    Camera(Cicada::Map& m) : map(m) {
        viewport.zoom = 1.0f;
        viewport.size = {1280.0f, 720.0f};
    }
    
    void setPosition(float x, float y) {
        viewport.position = {x, y};
        map.setViewport(viewport);
    }
    
    void zoom(float factor) {
        viewport.zoom = factor;
        // Adjust visible chunks
        map.setViewport(viewport);
    }
    
    auto getVisibleChunks() {
        return map.getVisibleChunks();
    }
};
```

### Creating a Tile Painter

```cpp
class TilePainter {
    Cicada::MapEditor editor;
    Cicada::TerrainBrush brush;
    
public:
    TilePainter(Cicada::Map& map) : editor(map) {
        brush.setSize(3);
        brush.setShape(Cicada::TerrainBrush::BrushShape::CIRCLE);
        editor.setBrush(brush);
    }
    
    void paintAt(int x, int y, Cicada::TileId tile_id) {
        brush.setTileId(tile_id);
        brush.setMode(Cicada::TerrainBrush::BrushMode::PAINT);
        editor.applyBrush({x, y});
    }
    
    void eraseAt(int x, int y) {
        brush.setMode(Cicada::TerrainBrush::BrushMode::ERASE);
        editor.applyBrush({x, y});
    }
    
    void undo() {
        if (editor.canUndo()) {
            editor.undo();
        }
    }
};
```

### Tile Property System

```cpp
void setupTileSet(Cicada::Map& map) {
    auto& tileset = map.getTileSet();
    
    // Grass tile
    Cicada::TileProperties grass;
    grass.name = "Grass";
    grass.collision_flags = 0;  // Walkable
    grass.texture_path = "assets/tiles/grass.png";
    tileset.registerTile(grass);
    
    // Stone tile
    Cicada::TileProperties stone;
    stone.name = "Stone";
    stone.collision_flags = 1;  // Solid
    stone.texture_path = "assets/tiles/stone.png";
    tileset.registerTile(stone);
    
    // Water tile (animated)
    Cicada::TileProperties water;
    water.name = "Water";
    water.collision_flags = 0;
    water.animated = true;
    water.animation_speed = 8;
    water.texture_path = "assets/tiles/water.png";
    tileset.registerTile(water);
}
```

### Collision Detection

```cpp
bool canMoveTo(Cicada::Map& map, int x, int y, Cicada::LayerId collision_layer) {
    auto tile = map.getTile(collision_layer, {x, y});
    if (!tile) {
        return false;  // Out of bounds
    }
    
    // Check collision flags
    constexpr uint16_t SOLID_FLAG = 1;
    return !tile->hasFlag(SOLID_FLAG);
}

// Usage
if (canMoveTo(map, new_x, new_y, collision_layer)) {
    player_x = new_x;
    player_y = new_y;
}
```

### Chunk Management

```cpp
void optimizeMemory(Cicada::Map& map, const Cicada::Viewport& viewport) {
    // Get all loaded chunks
    auto loaded = map.getLoadedChunks();
    
    // Unload chunks far from viewport
    for (const auto& chunk : loaded) {
        auto aabb = chunk->getAABB();
        
        // Simple AABB intersection test
        if (aabb.x + aabb.z < viewport.position.x - 1000 ||
            aabb.x > viewport.position.x + viewport.size.x + 1000) {
            // Unload chunk
            chunk->setState(Cicada::ChunkState::UNLOADING);
        }
    }
}
```

## Performance Tips

### For Large Maps
```cpp
// Adjust chunk size for large worlds
const int LARGE_CHUNK_SIZE = 64;  // Larger chunks, fewer total

// Limit visible area
viewport.size = {1280, 720};  // Don't over-render
```

### For Mobile Devices
```cpp
// Reduce memory footprint
chunk_manager.setMaxChunks(256);  // Fewer cached chunks

// Limit layer count
map.addLayer(...);  // Only essential layers
map.addLayer(...);
// Don't add too many layers
```

### For Procedural Generation
```cpp
// Pre-generate chunks in background thread
// Use async chunk loading
// Cache generated data in TileSet
```

## Troubleshooting

### Chunks not loading?
- Check viewport position and size
- Verify chunk culling logic
- Monitor memory usage

### Tiles not updating?
- Ensure dirty flag is set
- Check layer visibility
- Verify coordinate conversions

### Memory leaks?
- Call map.shutdown() in destructor
- Verify all layers are cleaned up
- Check for circular references

## See Also

- [GETTING_STARTED.md](GETTING_STARTED.md) - Quick start guide
- [ARCHITECTURE.md](ARCHITECTURE.md) - System design
- [README.md](README.md) - Full documentation
