# CicadaMapSystem - Industrial Grade 2D Game Engine Map System

![C++17](https://img.shields.io/badge/C%2B%2B-17-blue) ![License](https://img.shields.io/badge/License-MIT-green) ![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen)

A high-performance, production-ready 2D game engine map system built with C++17. Features multiple coordinate systems, dynamic chunk loading, advanced editing tools, and comprehensive performance optimization.

## 🎯 Key Features

### 🗺️ Multiple Map Types
- **Grid Maps** - Standard orthogonal tile grids  
- **Isometric Maps** - Isometric perspective projection
- **Hexagonal Maps** - Hex-based tile systems (pointy/flat-top)

### 🎨 Advanced Layer System
- Unlimited layers with independent properties
- Layer types: Background, Terrain, Object, Decoration, Collision, Custom
- Per-layer control: opacity, visibility, parallax, z-indexing
- Efficient dirty-flag-based rendering

### 📦 Intelligent Chunk System
- Configurable chunk size (default 32x32 tiles)
- Automatic LRU cache management (up to 1024 chunks)
- Dynamic loading/unloading based on viewport
- Efficient memory management with pooling

### ⚡ Performance Optimizations
- QuadTree spatial partitioning for collision detection
- Viewport culling for rendering optimization
- Real-time performance monitoring and profiling
- Batch operations for efficient bulk modifications

### 🎮 In-Game Editing
- Full undo/redo support with action stacks
- Multiple brush shapes: Circle, Square, Line
- Brush modes: Paint, Erase, Fill, Pick
- Copy/paste operations for terrain manipulation

### 🔧 Coordinate System Conversion
- Seamless conversion between world, chunk, and local coordinates
- Support for custom coordinate systems
- Screen to world coordinate mapping for mouse picking

### 📊 Comprehensive API
- TileSet management for tile definitions and properties
- Batch operations (fill rectangles, clear areas)
- Performance statistics and real-time monitoring
- Memory usage tracking and optimization

## 📋 Architecture

```
┌─────────────────────────────────────────┐
│         Game Application Layer          │
└────────────────────┬────────────────────┘
                     │
┌────────────────────▼────────────────────┐
│    Editor & Tools (Undo/Redo, Brush)    │
└────────────────────┬────────────────────┘
                     │
┌────────────────────▼────────────────────┐
│  Rendering Layer (OpenGL, SDL3)         │
└────────────────────┬────────────────────┘
                     │
┌────────────────────▼────────────────────┐
│  Core Map System                        │
│  ┌──────────┐  ┌────────────────────┐  │
│  │  Map     │  │  ChunkManager      │  │
│  │ Layers   │  │  (LRU Cache)       │  │
│  │ Tiles    │  │  Chunk Lifecycle   │  │
│  └──────────┘  └────────────────────┘  │
└────────────────────┬────────────────────┘
                     │
┌────────────────────▼────────────────────┐
│ Coordinate Systems & Utilities          │
│ Grid | Isometric | Hex | QuadTree       │
└─────────────────────────────────────────┘
```

## 🚀 Quick Start

### Requirements
- CMake 3.20+
- C++17 compatible compiler
- SDL3
- OpenGL 3.3+
- GLM (header-only)
- Catch2 3.x (for testing)

### Build

```bash
git clone https://github.com/XiaJack/CicadaMapSystem.git
cd CicadaMapSystem
mkdir build && cd build
cmake ..
cmake --build . --config Release
ctest  # Run tests
```

### Basic Usage

```cpp
#include "src/core/Map.h"
using namespace Cicada;

// Create map
MapConfig config;
config.type = MapType::GRID;
config.width = 512;
config.height = 512;
config.num_layers = 3;

Map map(config);
map.initialize();

// Add layers
LayerId terrain = map.addLayer(LayerType::TERRAIN, "Terrain", 0);

// Tile operations
map.setTile(terrain, {10, 20}, 42);      // Set tile
map.fillRect(terrain, {0, 0}, {50, 50}, 5);  // Fill area
map.clearRect(terrain, {25, 25}, {75, 75}); // Clear area

// Update viewport for rendering
Viewport viewport;
viewport.position = {0, 0};
viewport.size = {1280, 720};
map.setViewport(viewport);

// Get visible chunks for rendering
auto visible = map.getVisibleChunks();
```

## 📚 Documentation

- **[README.md](README.md)** - Full project documentation
- **[GETTING_STARTED.md](GETTING_STARTED.md)** - Quick start guide with examples
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Detailed system architecture
- **[EXAMPLES.md](EXAMPLES.md)** - Code examples and snippets

## 🧪 Testing

Comprehensive test suite with **40+ unit tests**:

```bash
# Run all tests
ctest

# Run specific category
./cicada_tests "[Map]"
./cicada_tests "[Layer]"
./cicada_tests "[Chunk]"
```

## 📈 Performance Characteristics

| Metric | Value |
|--------|-------|
| Memory per Tile | ~13 bytes |
| Memory per Chunk (32x32) | ~13.4 KB |
| Max Chunks in Memory | 1024 (configurable) |
| Max Chunk Pool | ~13.7 MB |
| Viewport Culling | Automatic |
| Supported Map Sizes | Infinite or up to 65K×65K tiles |

## 💡 Usage Examples

### Creating a Game World

```cpp
// Create map
Map map(MapConfig{...});
map.initialize();

// Add terrain and object layers
LayerId terrain = map.addLayer(LayerType::TERRAIN, "Terrain", 0);
LayerId objects = map.addLayer(LayerType::OBJECT, "Objects", 1);

// Fill with tiles
map.fillRect(terrain, {0, 0}, {511, 511}, 1);  // Grass
map.setTile(objects, {100, 100}, 50);          // Tree
```

### In-Game Editing

```cpp
MapEditor editor(map);

// Setup brush
TerrainBrush brush;
brush.setSize(5);
brush.setShape(TerrainBrush::BrushShape::CIRCLE);
brush.setMode(TerrainBrush::BrushMode::PAINT);
brush.setTileId(42);
editor.setBrush(brush);

// Apply and undo
editor.applyBrush({256, 256});
if (editor.canUndo()) {
    editor.undo();
}
```

### Coordinate System Usage

```cpp
// Grid coordinates
GridCoordinate grid;
auto screen_pos = grid.worldToScreen({10, 5});
auto world_pos = grid.screenToWorld({320, 160});

// Isometric coordinates
IsometricCoordinate iso;
auto iso_screen = iso.worldToScreen({5, 3});

// Hexagonal coordinates
HexCoordinate hex(HexCoordinate::HexLayout::POINTY_TOP);
auto hex_screen = hex.worldToScreen({3, 4});
```

### Performance Monitoring

```cpp
auto& perf = Performance::getInstance();

// In game loop
perf.beginFrame();
// ... game logic ...
perf.endFrame();

// Get statistics
printf("FPS: %u\n", perf.getFPS());
printf("Frame Time: %.2f ms\n", perf.getFrameTime() * 1000.0f);
```

## 🔧 Configuration

### Adjusting Chunk Manager

```cpp
ChunkManager manager(512);  // Max 512 chunks
```

### Custom Map Size

```cpp
MapConfig config;
config.width = 4096;
config.height = 4096;
config.infinite = true;  // Enable infinite map
```

### Layer Properties

```cpp
auto layer = map.getLayer(layer_id);
layer->setOpacity(200);      // 0-255
layer->setVisible(true);
layer->setParallax(0.5f, 0.5f);
layer->setZIndex(5);
```

## 🎮 Extensibility

### Custom Coordinate Systems

```cpp
class PerspectiveCoordinate : public CoordinateSystem {
    Coord2F worldToScreen(const Coord2D& world_pos) const override;
    Coord2D screenToWorld(const Coord2F& screen_pos) const override;
    MapType getType() const override { return MapType::GRID; }
};
```

### Custom Brushes

```cpp
class PerlinBrush : public TerrainBrush {
    void apply(Map& map, const Coord2D& center) override;
    // Custom implementation
};
```

### Custom Renderers

```cpp
class VulkanRenderer : public MapRenderer {
    bool initialize(uint32_t width, uint32_t height) override;
    void renderMap(const Map& map, const Viewport& viewport) override;
    // Custom implementation
};
```

## 📊 Project Statistics

- **Lines of Code**: ~5,000+
- **Header Files**: 20+
- **Implementation Files**: 20+
- **Test Files**: 10+
- **Documentation Pages**: 5
- **Code Examples**: 4+
- **Unit Tests**: 40+

## 🗺️ Roadmap

- [ ] Async chunk loading
- [ ] Map serialization (JSON/Binary)
- [ ] Streaming from disk
- [ ] Physics integration
- [ ] Animation system
- [ ] Advanced culling (BSP, Portals)
- [ ] Procedural generation utilities
- [ ] Multiplayer synchronization
- [ ] Map editor tool

## 🤝 Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Submit a pull request

## 📄 License

This project is provided as-is for production use.

## 🎓 Learning Resources

1. Start with [GETTING_STARTED.md](GETTING_STARTED.md)
2. Review [EXAMPLES.md](EXAMPLES.md) for code examples
3. Read [ARCHITECTURE.md](ARCHITECTURE.md) for design details
4. Explore test files for usage patterns
5. Check out complete examples in `example/` directory

## 🏆 Production Ready

✅ Comprehensive test coverage  
✅ Memory-safe implementation  
✅ Performance optimized  
✅ Well-documented API  
✅ Multiple use case examples  
✅ Extensible architecture  

## 📞 Support

For questions or issues:
- Create a GitHub Issue
- Check documentation
- Review test cases
- Explore example code

---

**CicadaMapSystem** - Professional 2D Game Engine Map Solution  
Built with C++17 • Optimized for Performance • Ready for Production
