# Pie in the Sky (PI) 3D Graphics Engine Specification

**Author**: Mark Bednarczyk  
**Date**: May 31, 2025  
**Version**: 1.0  

## Version History
| **Version** | **Date**       | **Changes**                                                                 |
|-------------|----------------|-----------------------------------------------------------------------------|
| 1.0         | May 31, 2025   | Initial deployment with core modules (`pi-engine-core`, `pi-engine-math`, `pi-engine-util`). Includes scene management, plugin system, app lifecycle, multi-threading with StructuredTaskScope, and basic OpenGL rendering. Plugin modules for advanced rendering, physics, audio, UI, and editor are planned. |

## 1. Overview
The Pie in the Sky (PI) engine is a next-generation 3D graphics engine designed to combine the simplicity of HTML/CSS with the power of modern game engines like Unreal Engine. It targets indie developers, educators, and content creators, offering a modular, community-driven architecture, a custom editor, and advanced features like AI-driven tools and real-time streaming. The engine is cross-platform, supporting Windows, macOS, Linux, iOS, Android, consoles (PS5, Xbox), VR/AR, and web via WebGL/WASM.

### Objectives
- **Simplicity**: YAML-based configurations and a clean editor UI ensure ease of use.
- **Power**: Match/exceed Unreal Engine features with plugins for rendering, physics, audio, and more.
- **Extensibility**: Fully modular plugin system enables community innovation.
- **Performance**: Multi-threaded pipeline using StructuredTaskScope and Virtual Threads leverages modern hardware.
- **Accessibility**: First-class Java API and guided workflows lower the entry barrier.

### Key Features
- Custom `.pi?` file formats (e.g., `.pis` for scenes, `.pio` for objects).
- Modular architecture with plugins for all components, loaded/unloaded per app.
- Custom PI-based editor for scene development, app management, plugins, marketplace, assets, and configs.
- Multi-threaded (MT) pipeline using Java’s StructuredTaskScope and Virtual Threads.
- Pluggable rendering backends (Vulkan, OpenGL, DirectX, etc.).
- First-class Java API, with a separate C/C++ API planned for future compatibility.
- Support for external asset formats (FBX, glTF, OBJ, COLLADA).
- Feature parity with Unreal Engine, enhanced by AI and streaming.

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

### 2.2. Module Overview
The PI engine is organized into a set of Java modules, grouped into two categories: **Base Modules** and **Plugin Modules**. Base Modules provide the foundational functionality, while Plugin Modules add or expand specific features.

#### Base Modules
| **Module**       | **Maven Artifact ID** | **Group ID**   | **Purpose**                                                                 |
|------------------|-----------------------|----------------|-----------------------------------------------------------------------------|
| `org.piengine`   | `pi-engine`           | `org.piengine` | The parent module, aggregating all submodules for the PI engine project.   |
| `org.piengine.math` | `pi-engine-math`      | `org.piengine` | Provides mathematical foundations, including coordinates (`Cartesian3D`, `Vector3f`), shapes (`Point3D`), matrices, and math functions (`MathUtils`). |
| `org.piengine.util` | `pi-engine-util`      | `org.piengine` | Offers general-purpose utilities, including concurrency (`UpgradableReadWriteLock`, `Lockable`), event handling (`Omnibus`), and registration (`Registration`). |
| `org.piengine.core` | `pi-engine-core`      | `org.piengine` | The main module providing the core API and functionality, including scene management (`Scene`, `SceneNode`), plugin system (`Plugin`), apps (`App`), task scheduling, input handling, and profiles. |

#### Plugin Modules
| **Module**                   | **Maven Artifact ID**   | **Group ID**   | **Purpose**                                                                 |
|------------------------------|-------------------------|----------------|-----------------------------------------------------------------------------|
| `org.piengine.render.opengl` | `pi-engine-render-opengl` | `org.piengine` | OpenGL rendering backend for the PI engine (planned).                      |
| `org.piengine.physics.bullet`| `pi-engine-physics-bullet`| `org.piengine` | Bullet Physics backend for physics simulation (planned).                   |
| `org.piengine.audio.openal`  | `pi-engine-audio-openal`| `org.piengine` | OpenAL audio backend for 2D and 3D spatial audio (planned).                |
| `org.piengine.ui.imgui`      | `pi-engine-ui-imgui`  | `org.piengine` | ImGui-based UI backend for the custom editor (planned).                    |
| `org.piengine.editor`        | `pi-engine-editor`    | `org.piengine` | Custom editor for scene development, app management, and plugins (planned). |

### 2.3. Modular Design
The engine is organized into base and plugin modules. Base modules (`org.piengine.math`, `org.piengine.util`, `org.piengine.core`) provide the core functionality. The plugin modules add features like rendering, physics, audio, UI, and editor functionality.

### 2.4. Plugin System
Plugins are Java classes packaged as `.pip` files (ZIP-compressed JARs), loaded via `ServiceLoader`. They are loaded/unloaded per app, enabling runtime customization. Plugins support pausing/unpausing and initialization for specific apps. Supported components:
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
package org.piengine.core.plugin;
public interface Plugin {
    void init();
    void initForApp(App app);
    void update(float dt);
    void pause();
    void unpause();
    void shutdown();
    void processScene(Scene scene);
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

### 2.5. Apps
Apps (e.g., `MenuApp`, `GameApp`) define runtime contexts, each with specific scenes, plugins, and scripts. Apps are configured in YAML and managed via the editor.

**Example**:
```yaml
apps:
  - id: MenuApp
    scene: menu.pis
    plugins: [basic_raster.pip, imgui_ui.pip]
  - id: GameApp
    scene: level1.pis
    plugins: [ray_tracing.pip, physx.pip, twitch_stream.pip]
```

### 2.6. Multi-Threaded Pipeline
The engine uses Java’s **StructuredTaskScope** and **Virtual Threads** (JDK 23) for a scalable, debuggable MT pipeline. Components include:
- **MT Raster Pipeline**: Parallelize culling, draw calls, post-processing.
- **MT Animation Pipeline**: Parallelize skeleton updates, skinning.
- **MT Event Handling**: Dispatch input and network events concurrently.
- **MT Apps**: Update UI and app transitions in parallel.
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

### 2.7. File Formats
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

### 2.8. Coordinate and Shape System
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

### 2.9. Rendering Backends
The raster pipeline is pluggable via plugins, supporting:
- Vulkan (planned primary, cross-platform).
- OpenGL (MVP default, fallback for older hardware).
- DirectX, Metal, WebGL (via community plugins).

**Example**:
```java
package org.piengine.core.plugin;
public interface RasterPlugin extends Plugin {
    void draw(SceneNode node);
}
```

### 2.10. APIs
The Java API is the primary interface, designed to be portable for a future C/C++ API.

**Example**:
```java
import org.piengine.core.scene.Scene;
import org.piengine.core.scene.SceneNode;
Scene scene = new Scene();
SceneNode node = scene.createNode("cube1");
node.setMesh("cube.pio");
```

## 3. Custom PI-Based Editor

### 3.1. Overview
The editor, built as an app (`EditorApp`), provides a simple, powerful interface for scene development, app management, plugin handling, marketplace interaction, asset management, and config profiles.

### 3.2. Design Principles
- **Simplicity**: Clean, modular UI with drag-and-drop, visual scripting, and templates.
- **Power**: Plugin-driven extensibility for procedural tools, AI, and real-time previews.
- **Accessibility**: Guided workflows, tooltips, and beginner mode.
- **Modularity**: Editor features are plugins, allowing community enhancements.

### 3.3. Features
1. **Scene Development**:
   - Manual editing, procedural generation, importing (FBX, OBJ, glTF, COLLADA).
   - Real-time preview using `RasterPlugin`.
2. **App Management**:
   - Visual editor for apps with flowchart transitions.
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

### 3.5. Editor as App
```yaml
apps:
  - id: EditorApp
    scene: editor.pis
    plugins: [scene_editor.pip, asset_import.pip, marketplace.pip]
```

## 4. Feature Comparison
| **Feature**                  | **Unreal Engine**                              | **PI’s Approach**                                                                 |
|------------------------------|-----------------------------------------------|----------------------------------------------------------------------------------|
| **Rendering**                | PBR, ray tracing, Vulkan/Metal                | Core: Basic OpenGL. Plugins: Vulkan, DirectX, ray tracing, AI-optimized shaders  |
| **Physics**                  | PhysX, Chaos                                  | Core: None. Plugins: Bullet, PhysX, Havok, GPU-accelerated physics              |
| **Audio**                    | 3D audio, reverb, occlusion                   | Core: Basic 2D audio. Plugins: OpenAL, FMOD, AI-driven sound propagation        |
| **Animation**                | Skeletal, blend spaces, IK                    | Core: Basic keyframing. Plugins: Skeletal, procedural, ML-driven animation      |
| **Networking**               | Replication, rollback netcode                 | Core: Basic sockets. Plugins: Lockstep, rollback, WebSocket, P2P                |
| **Streaming**                | Limited (external tools)                      | Core: None. Plugins: FFmpeg, Twitch, YouTube, OBS integration                   |
| **Input**                    | Keyboard, gamepad, VR                         | Core: Basic keyboard/mouse. Plugins: Gamepad, VR, custom devices                |
| **Particles**                | Niagara (GPU-driven)                          | Core: Basic sprites. Plugins: GPU particles, procedural effects, ML-driven      |
| **Environment**              | Procedural terrain, foliage                   | Core: Static meshes. Plugins: Procedural terrain, voxel-hybrid, AI-generated    |
| **Styling**                  | PBR materials, UMG UI                         | Core: Basic materials. Plugins: PBR materials, YAML-defined UI, ImGui           |
| **Editor**                   | World Outliner, Blueprints, complex UI        | Custom PI-based, simple UI, procedural tools, marketplace, profiles             |
| **VR/AR**                    | OpenXR, VR/AR support                        | Core: None. Plugins: OpenXR, ARKit/ARCore, spatial computing                   |
| **AI**                       | Behavior trees, navmeshes, EQS                | Core: None. Plugins: ML-driven AI, behavior trees, generative NPCs             |

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
| **Module**                     | **Maven Artifact ID**     | **Group ID**         | **Package**                           | **Purpose**                                                                 |
|--------------------------------|---------------------------|----------------------|---------------------------------------|-----------------------------------------------------------------------------|
| `org.piengine`                 | `pi-engine`               | `org.piengine`       | N/A                                   | Parent module aggregating all submodules.                                  |
| `org.piengine.math`            | `pi-engine-math`          | `org.piengine`       | `org.piengine.commons.math`           | Coordinates, matrices, shapes, math functions.                             |
| `org.piengine.util`            | `pi-engine-util`          | `org.piengine`       | `org.piengine.commons.util`           | Concurrent locks, Registration, event bus (Omnibus), utilities.            |
| `org.piengine.core`            | `pi-engine-core`          | `org.piengine`       | `org.piengine.core`                   | Plugin management, scenegraphs, apps, task scheduling, input, profiles. |
| `org.piengine.render.opengl`   | `pi-engine-render-opengl` | `org.piengine`       | `org.piengine.render.opengl`          | OpenGL rendering (planned).                                                |
| `org.piengine.physics.bullet`  | `pi-engine-physics-bullet`| `org.piengine`       | `org.piengine.physics.bullet`         | Bullet Physics (planned).                                                  |
| `org.piengine.audio.openal`    | `pi-engine-audio-openal`  | `org.piengine`       | `org.piengine.audio.openal`           | OpenAL audio (planned).                                                    |
| `org.piengine.ui.imgui`        | `pi-engine-ui-imgui`      | `org.piengine`       | `org.piengine.ui.imgui`               | ImGui-based editor UI (planned).                                           |
| `org.piengine.editor`          | `pi-engine-editor`        | `org.piengine`       | `org.piengine.editor`                 | Custom editor (planned).                                                   |

**Module Descriptors**:
```java
module org.piengine.math {
    exports org.piengine.commons.math;
    exports org.piengine.commons.math.coordinates;
    exports org.piengine.commons.math.shapes;
    requires org.yaml.snakeyaml;
    requires java.base;
}

module org.piengine.util {
    exports org.piengine.commons.util;
    exports org.piengine.commons.util.eventbus;
    exports org.piengine.commons.util.concurrent.locks;
    requires java.base;
}

module org.piengine.core {
    exports org.piengine.core;
    exports org.piengine.core.plugin;
    exports org.piengine.core.scene;
    exports org.piengine.core.app;
    exports org.piengine.core.task;
    exports org.piengine.core.input;
    exports org.piengine.core.profile;
    requires org.piengine.math;
    requires org.piengine.util;
    requires org.yaml.snakeyaml;
    requires java.base;
}

module org.piengine.render.opengl {
    exports org.piengine.render.opengl;
    requires org.piengine.core;
    requires org.piengine.math;
    requires org.lwjgl.opengl;
}
```

**Maven Artifacts**:
- **POM Example (pi-engine-math)**:
  ```xml
  <groupId>org.piengine</groupId>
  <artifactId>pi-engine-math</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <dependencies>
      <dependency>
          <groupId>org.yaml.snakeyaml</groupId>
          <artifactId>snakeyaml</artifactId>
      </dependency>
  </dependencies>
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
- **Commons**: `pi-engine-math` (coordinates, shapes, math), `pi-engine-util` (locks, Registration, Omnibus).
- **Core**: Scenegraphs (`.pis`), input handling, plugin manager, task scheduling, profiles.
- **Editor**: Scene editing, app configuration, asset management, plugin enable/disable (planned).
- **Plugins**: `VulkanRasterPlugin`, `BulletPhysicsPlugin`, `OpenALAudioPlugin`, `ImGuiUIPlugin`, `SceneEditorPlugin` (planned).
- **Formats**: `.pis`, `.pio`, `.pim`, `.pib`.

## 9. Challenges and Mitigations
- **Performance**: Optimize with LWJGL bindings.
- **Module Dependencies**: Provide `pi-engine-mvp` bundle.
- **Maven Setup**: Automate with GitHub Actions.

## 10. Future Enhancements
- C/C++ API, AI tools, cloud rendering, collaborative editing, scripting.

## 11. Glossary
- **.pi?**: File extensions (e.g., `.pis`).
- **Plugin**: `.pip` JAR.
- **App**: Runtime context.
- **StructuredTaskScope**: Structured concurrency.
- **Cartesian3D**: 3D coordinate interface.

## 12. References
- Unreal Engine, Godot, O3DE.
- LWJGL, imgui-java, jassimp.
- Java 23 (JEP 462).
- SnakeYAML.