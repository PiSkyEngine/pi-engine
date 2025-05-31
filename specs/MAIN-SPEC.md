# Pie in the Sky (PI) 3D Graphics Engine Specification

**Author**: Mark Bednarczyk  
**Date**: May 29, 2025  
**Version**: 1.2  

## Version History
| **Version** | **Date**       | **Changes**                                                                 |
|-------------|----------------|-----------------------------------------------------------------------------|
| 1.0         | May 22, 2025   | Initial version with updated coordinate system, `org.piengine` group ID, JDK 23, and SnakeYAML module support. |
| 1.1         | May 29, 2025   | Refactored `pi-engine-core` into `pi-commons-math`, `pi-commons-utils`, and `pi-engine-core`. Added independent commons modules for jMonkeyEngine compatibility. Updated module structure and dependencies. |
| 1.2         | May 29, 2025   | Renamed commons modules to `pi-engine-commons-*`, split `pi-engine-commons-math` into `*.coordinates`, `*.math`, `*.shapes` packages, moved `core.util` to `pi-engine-commons-utils`, added Omnibus event bus under `*.utils.eventbus`. |

## 1. Overview
The Pie in the Sky (PI) engine is a next-generation 3D graphics engine designed to combine the simplicity of HTML/CSS with the power of modern game engines like Unreal and Unity. It targets indie developers, educators, and content creators, offering a modular, community-driven architecture, a custom editor, and advanced features like AI-driven tools and real-time streaming. The engine is cross-platform, supporting Windows, macOS, Linux, iOS, Android, consoles (PS5, Xbox), VR/AR, and web via WebGL/WASM.

### Objectives
- **Simplicity**: YAML-based configurations and a clean editor UI ensure ease of use.
- **Power**: Match/exceed Unreal and jMonkeyEngine features with plugins for rendering, physics, audio, and more.
- **Extensibility**: Fully modular plugin system enables community innovation.
- **Performance**: Multi-threaded pipeline using StructuredTaskScope and Virtual Threads leverages modern hardware.
- **Accessibility**: First-class Java API and guided workflows lower the entry barrier.

### Key Features
- Custom `.pi?` file formats (e.g., `.pis` for scenes, `.pio` for objects).
- Modular architecture with plugins for all components, loaded/unloaded per app state.
- Custom PI-based editor for scene development, app state management, plugins, marketplace, assets, and configs.
- Multi-threaded (MT) pipeline using Java’s StructuredTaskScope and Virtual Threads.
- Pluggable rendering backends (Vulkan, OpenGL, DirectX, etc.).
- First-class Java API, with a separate C/C++ API planned for future compatibility.
- Independent commons modules (`pi-engine-commons-math`, `pi-engine-commons-utils`) for use in other projects (e.g., jMonkeyEngine).
- Support for external asset formats (FBX, glTF, OBJ, COLLADA).
- Feature parity with Unreal/jMonkeyEngine, enhanced by AI and streaming.

## 2. Core Architecture

### 2.1. Java-First Implementation
The PI engine is implemented entirely in Java (version 23+), leveraging Java bindings for native libraries to achieve near-native performance. A future C/C++ API will mirror the Java API for compatibility.

- **Native Libraries**:
  - **LWJGL**: OpenGL, Vulkan, Bullet Physics, OpenAL bindings.
  - **imgui-java**: ImGui for editor UI.
  - **Assimp**: Asset imports (FBX, OBJ, glTF, COLLADA) via jassimp.
  - **SnakeYAML**: YAML parsing for `.pi?` files and configs (`org.yaml.snakeyaml`).
- **Java Features**: Records, sealed classes, virtual threads, StructuredTaskScope (JDK 23).
- **C/C++ Compatibility**: Java API uses portable abstractions (e.g., `SceneNode.setMesh(String)`) to align with future C/C++ API (e.g., `SceneNode::setMesh(std::string)`).

### 2.2. Modular Design
The engine is organized into commons and engine-specific modules. Commons modules (`pi-engine-commons-math`, `pi-engine-commons-utils`) provide general-purpose types, independent for use in other projects (e.g., jMonkeyEngine). The core module (`pi-engine-core`) handles engine-specific APIs, with additional modules for rendering, physics, audio, UI, and editor.

### 2.3. Plugin System
Plugins are Java classes packaged as `.pip` files (ZIP-compressed JARs), loaded via `ServiceLoader`. They are loaded/unloaded per app state, enabling runtime customization. Supported components:
- **Raster Pipeline**: Shaders, ray tracing, post-processing.
- **Physics**: Rigid/soft bodies, collisions.
- **Audio**: 3D spatial audio, effects.
- **Animation**: Skeletal, procedural, inverse kinematics.
- **Styling**: Materials, UI theming.
- **Particles/Smoke/Fire**: GPU-driven effects.
- **Environment**: Terrain, foliage, skyboxes.
- **Networking**: Client-server, P2P, rollback netcode.
- **Streaming**: Twitch, YouTube, OBS integration.
- **Input-Controller**: Keyboard, gamepad, VR, custom devices.

**Plugin Interface Example**:
```java
package org.pi.game.engine.core.plugin;
public interface Plugin {
    void init();
    void update(float dt);
    void shutdown();
}
public class VulkanRasterPlugin implements Plugin, RasterPlugin { ... }
```

**Plugin Config Example**:
```yaml
plugins:
  - id: vulkan_raster
    path: vulkan.pip
    priority: 1
  - id: physx
    path: physx.pip
```

### 2.4. Application States
App states (e.g., `MenuState`, `GameState`) define runtime contexts, each with specific scenes, plugins, and scripts. States are configured in YAML and managed via the editor.

**Example**:
```yaml
app_states:
  - id: MenuState
    scene: menu.pis
    plugins: [basic_raster.pip, imgui_ui.pip]
  - id: GameState
    scene: level1.pis
    plugins: [ray_tracing.pip, physx.pip, twitch_stream.pip]
```

### 2.5. Multi-Threaded Pipeline
The engine uses Java’s **StructuredTaskScope** and **Virtual Threads** (JDK 23) for a scalable, debuggable MT pipeline. Components include:
- **MT Raster Pipeline**: Parallelize culling, draw calls, post-processing.
- **MT Animation Pipeline**: Parallelize skeleton updates, skinning.
- **MT Event Handling**: Dispatch input and network events concurrently.
- **MT Application States**: Update UI and state transitions in parallel.
- **MT Plugin Tasks**: Execute plugin tasks within task scopes.

**Example Usage**:
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<RenderResult> renderFuture = scope.fork(() -> rasterPlugin.draw(scene));
    Future<PhysicsResult> physicsFuture = scope.fork(() -> physicsPlugin.simulate(scene));
    scope.join().throwIfFailed();
    return new FrameResult(renderFuture.resultNow(), physicsFuture.resultNow());
} catch (InterruptedException | ExecutionException e) {
    throw new RuntimeException("Pipeline failed", e);
}
```

**Config Example**:
```yaml
pipeline:
  max_tasks: 1000
  shutdown_policy: on_failure
  use_virtual_threads: true
```

### 2.6. File Formats
Custom `.pi?` formats use a consistent prefix for discoverability (e.g., `*.pi*` search). Formats are YAML-based or binary, with extensibility via metadata.

| **Extension** | **Purpose**             | **Example Content**                     |
|---------------|-------------------------|-----------------------------------------|
| `.pis`        | Scenegraph              | Nodes, transforms, references to `.pio` |
| `.pio`        | Object (mesh)           | Vertices, indices, normals             |
| `.pim`        | Material                | Shaders, textures, properties          |
| `.pia`        | Animation               | Keyframes, bones                       |
| `.pit`        | Texture                 | Image data, format                     |
| `.pip`        | Plugin                  | ZIP-compressed JAR                     |
| `.pib`        | Asset Bundle            | Collection of `.pio`, `.pim`, etc.     |

**Scene Example**:
```yaml
scene:
  version: 1.0
  nodes:
    - id: cube1
      type: mesh
      geometry: cube.pio
      material: shiny_metal.pim
      transform:
        position: [0, 0, 0]
```

### 2.7. Coordinate and Shape System
- **Coordinates** (in `org.piengine.commons.math.coordinates`):
  - **Root Interface**: `Coordinate` (shapeless, defines dimension and validity).
  - **Cartesian3D**: Base for 3D coordinates, with inner interfaces `Cartesian3f`, `Cartesian3d` for float and double precision.
  - **Vector3f**: Extends `Cartesian3D` for vector operations (e.g., normalize, cross, dot), supports matrix operations via `Matrix1x3`.
  - **Precision**: Defines precision types (`FLOAT`, `DOUBLE`) and conversions (in `org.piengine.commons.math`).
- **Shapes** (in `org.piengine.commons.math.shapes`):
  - **Point3D**: Represents a point shape, providing a `Cartesian3D` origin, with implementations `Point3f`, `Point3d`.
  - Planned: `Box`, `Circle`, `Sphere` for collision detection and procedural generation.
- **Math Functions** (in `org.piengine.commons.math`): Trigonometry, quaternions, interpolation via `MathUtils`.

**Example**:
```java
import org.piengine.commons.math.coordinates.Cartesian3D;
import org.piengine.commons.math.coordinates.Vector3f;
import org.piengine.commons.math.shapes.Point3D;
Cartesian3D origin = Point3D.ofFloat(1.0f, 2.0f, 3.0f).getOrigin();
Vector3f vector = Vector3f.ofFloat(0.0f, 1.0f, 0.0f);
```

### 2.8. Rendering Backends
The raster pipeline is pluggable via plugins, supporting:
- Vulkan (planned primary, cross-platform).
- OpenGL (MVP default, fallback for older hardware).
- DirectX, Metal, WebGL (via community plugins).

**Example**:
```java
package org.pi.game.engine.core.plugin;
public interface RasterPlugin extends Plugin {
    void draw(SceneNode node);
}
```

### 2.9. APIs
The Java API is the primary interface, designed to be portable for a future C/C++ API. Commons modules (`pi-engine-commons-math`, `pi-engine-commons-utils`) provide reusable types for other projects (e.g., jMonkeyEngine).

**Example**:
```java
import org.pi.game.engine.core.scene.Scene;
import org.pi.game.engine.core.scene.SceneNode;
Scene scene = new Scene();
SceneNode node = scene.createNode("cube1");
node.setMesh("cube.pio");
```

**jMonkeyEngine Integration**:
```java
import org.piengine.commons.math.coordinates.Cartesian3f;
import com.jme3.math.Vector3f;
Cartesian3f point = new Cartesian3f(1.0f, 2.0f, 3.0f);
Vector3f jmePoint = new Vector3f(point.xf(), point.yf(), point.zf());
```

## 3. Custom PI-Based Editor

### 3.1. Overview
The editor, built as an app state (`EditorState`), provides a simple, powerful interface for scene development, app state management, plugin handling, marketplace interaction, asset management, and config profiles.

### 3.2. Design Principles
- **Simplicity**: Clean, modular UI with drag-and-drop, visual scripting, and templates.
- **Power**: Plugin-driven extensibility for procedural tools, AI, and real-time previews.
- **Accessibility**: Guided workflows, tooltips, and beginner mode.
- **Modularity**: Editor features are plugins, allowing community enhancements.

### 3.3. Features
1. **Scene Development**:
   - Manual editing, procedural generation, importing (FBX, OBJ, glTF, COLLADA).
   - Real-time preview using `RasterPlugin`.
2. **App State Management**:
   - Visual editor for states with flowchart transitions.
3. **Plugin Management**:
   - Enable/disable, configure, update plugins.
4. **Marketplace Interaction**:
   - Browse plugins/assets at `marketplace.piengine.org` (planned).
5. **Asset Management**:
   - Import/export assets, `.pib` bundles, library with search.
6. **Config Management**:
   - Profiles for development, QA, alpha/beta, public release, tournaments.

### 3.4. UI Design
- Modular, dockable panels (inspired by Visual Studio Code).
- Beginner mode, visual scripting, embedded tutorials.
- Plugin: `ImGuiUIPlugin`.

### 3.5. Editor as App State
```yaml
app_states:
  - id: EditorState
    scene: editor.pis
    plugins: [scene_editor.pip, asset_import.pip, marketplace.pip]
```

## 4. Feature Comparison
| **Feature**                  | **Unreal Engine**                              | **jMonkeyEngine**                          | **PI’s Approach**                                                                 |
|------------------------------|-----------------------------------------------|-------------------------------------------|----------------------------------------------------------------------------------|
| **Rendering**                | PBR, ray tracing, Vulkan/Metal                | PBR, OpenGL, shaders                      | Core: Basic OpenGL. Plugins: Vulkan, DirectX, ray tracing, AI-optimized shaders  |
| **Physics**                  | PhysX, Chaos                                  | jBullet, Minie                            | Core: None. Plugins: Bullet, PhysX, Havok, GPU-accelerated physics              |
| **Audio**                    | 3D audio, reverb, occlusion                   | OpenAL, jmePhonon                         | Core: Basic 2D audio. Plugins: OpenAL, FMOD, AI-driven sound propagation        |
| **Animation**                | Skeletal, blend spaces, IK                    | Skeletal, basic IK                        | Core: Basic keyframing. Plugins: Skeletal, procedural, ML-driven animation      |
| **Networking**               | Replication, rollback netcode                 | SpiderMonkey, Netty                       | Core: Basic sockets. Plugins: Lockstep, rollback, WebSocket, P2P                |
| **Streaming**                | Limited (external tools)                      | None                                      | Core: None. Plugins: FFmpeg, Twitch, YouTube, OBS integration                   |
| **Input**                    | Keyboard, gamepad, VR                         | Keyboard, mouse, gamepad                  | Core: Basic keyboard/mouse. Plugins: Gamepad, VR, custom devices                |
| **Particles**                | Niagara (GPU-driven)                          | Particle Monkey, Effekseer                | Core: Basic sprites. Plugins: GPU particles, procedural effects, ML-driven      |
| **Environment**              | Procedural terrain, foliage                   | TerraMonkey, Blocks                       | Core: Static meshes. Plugins: Procedural terrain, voxel-hybrid, AI-generated    |
| **Styling**                  | PBR materials, UMG UI                         | Lemur, Nifty GUI                          | Core: Basic materials. Plugins: PBR materials, YAML-defined UI, ImGui           |
| **Editor**                   | World Outliner, Blueprints, complex UI        | jMonkey SDK, basic                        | Custom PI-based, simple UI, procedural tools, marketplace, profiles             |
| **VR/AR**                    | OpenXR, VR/AR support                        | Oculus Rift (limited)                     | Core: None. Plugins: OpenXR, ARKit/ARCore, spatial computing                   |
| **AI**                       | Behavior trees, navmeshes, EQS                | Basic pathfinding                         | Core: None. Plugins: ML-driven AI, behavior trees, generative NPCs             |

## 5. Asset Pipeline
- **Import**: FBX, OBJ, glTF, COLLADA via `AssetImportPlugin`.
- **Export**: `.pio`, `.pim`, `.pit`, `.pib` bundles.
- **Library**: Editor UI for asset organization, tagging, search.

## 6. Cross-Platform Support
- **Platforms**: Windows, macOS, Linux, iOS, Android, consoles, VR/AR, web.
- **Rendering**: Vulkan (planned), OpenGL (MVP), DirectX/Metal/WebGL (plugins).
- **Deployment**: Profile-based builds.

## 7. Implementation Details

### 7.1. Java Modules
| **Module**                     | **Group ID**         | **Package**                           | **Purpose**                                                                 |
|--------------------------------|----------------------|---------------------------------------|-----------------------------------------------------------------------------|
| `pi-engine-commons-math`       | `org.piengine`       | `org.piengine.commons.math`          | Coordinates, matrices, shapes, math functions                              |
| `pi-engine-commons-utils`      | `org.piengine`       | `org.piengine.commons.utils`         | Concurrent locks, Registration, event bus (Omnibus), utilities             |
| `pi-engine-core`               | `org.piengine`       | `org.pi.game.engine.core`            | Plugin management, scenegraphs, app states, task scheduling, input, profiles |
| `pi-engine-render-opengl`      | `org.piengine`       | `org.pi.game.engine.render.opengl`   | OpenGL rendering (planned)                                                 |
| `pi-engine-physics-bullet`     | `org.piengine`       | `org.pi.game.engine.physics.bullet`  | Bullet Physics (planned)                                                   |
| `pi-engine-audio-openal`       | `org.piengine`       | `org.pi.game.engine.audio.openal`    | OpenAL audio (planned)                                                     |
| `pi-engine-ui-imgui`           | `org.piengine`       | `org.pi.game.engine.ui.imgui`        | ImGui-based editor UI (planned)                                            |
| `pi-engine-editor`             | `org.piengine`       | `org.pi.game.engine.editor`          | Custom editor (planned)                                                    |

**Module Descriptors**:
```java
module pi.engine.commons.math {
    exports org.piengine.commons.math;
    exports org.piengine.commons.math.coordinates;
    exports org.piengine.commons.math.shapes;
    requires org.yaml.snakeyaml;
    requires java.base;
}

module pi.engine.commons.utils {
    exports org.piengine.commons.utils;
    exports org.piengine.commons.utils.eventbus;
    requires java.base;
}

module pi.game.engine.core {
    exports org.pi.game.engine.core;
    exports org.pi.game.engine.core.plugin;
    exports org.pi.game.engine.core.scene;
    exports org.pi.game.engine.core.state;
    exports org.pi.game.engine.core.task;
    exports org.pi.game.engine.core.input;
    exports org.pi.game.engine.core.profile;
    requires pi.engine.commons.math;
    requires pi.engine.commons.utils;
    requires org.yaml.snakeyaml;
    requires java.base;
}

module pi.game.engine.render.opengl {
    exports org.pi.game.engine.render.opengl;
    requires pi.game.engine.core;
    requires pi.engine.commons.math;
    requires org.lwjgl.opengl;
}
```

**Maven Artifacts**:
- **POM Example (pi-engine-commons-math)**:
  ```xml
  <groupId>org.piengine</groupId>
  <artifactId>pi-engine-commons-math</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <dependencies>
      <dependency>
          <groupId>org.yaml.snakeyaml</groupId>
          <artifactId>snakeyaml</artifactId>
          <version>2.3</version>
      </dependency>
  </dependencies>
  <properties>
      <maven.compiler.release>23</maven.compiler.release>
  </properties>
  ```

### 7.2. Core Libraries
- **Rendering**: LWJGL (OpenGL, Vulkan planned).
- **Physics**: LWJGL (Bullet planned).
- **Audio**: LWJGL (OpenAL planned).
- **UI**: imgui-java.
- **Assets**: jassimp, custom `.pib` bundler.
- **YAML**: SnakeYAML.

### 7.3. Multi-Threading
- **StructuredTaskScope**: Manages parallel tasks (JDK 23).
- **Virtual Threads**: Lightweight concurrency.
- **Debugging**: Simplified with StructuredTaskScope.

### 7.4. Community Ecosystem
- **Open-Source**: MIT license, GitHub (`PiSkyEngine/pisky-engine`).
- **Marketplace**: `marketplace.piengine.org` (planned).
- **SDK**: Plugin templates, documentation.

## 8. Minimum Viable Product (MVP)
- **Commons**: `pi-engine-commons-math` (coordinates, shapes, math), `pi-engine-commons-utils` (locks, Registration, Omnibus).
- **Core**: Scenegraphs (`.pis`), input handling, plugin manager, task scheduling, profiles.
- **Editor**: Scene editing, app state configuration, asset management, plugin enable/disable (planned).
- **Plugins**: `VulkanRasterPlugin`, `BulletPhysicsPlugin`, `OpenALAudioPlugin`, `ImGuiUIPlugin`, `SceneEditorPlugin` (planned).
- **Formats**: `.pis`, `.pio`, `.pim`, `.pib`.

## 9. Challenges and Mitigations
- **Performance**: Optimize with LWJGL bindings.
- **Module Dependencies**: Provide `pi-engine-mvp` bundle.
- **jMonkeyEngine Compatibility**: Ensure `pi-engine-commons-math` converts to `Vector3f`.
- **Maven Setup**: Automate with GitHub Actions.

## 10. Future Enhancements
- C/C++ API, AI tools, cloud rendering, collaborative editing, scripting.

## 11. Glossary
- **.pi?**: File extensions (e.g., `.pis`).
- **Plugin**: `.pip` JAR.
- **App State**: Runtime context.
- **StructuredTaskScope**: Structured concurrency.
- **Cartesian3D**: 3D coordinate interface.

## 12. References
- Unreal Engine, jMonkeyEngine, Godot, O3DE.
- LWJGL, imgui-java, jassimp.
- Java 23 (JEP 462).
- SnakeYAML.