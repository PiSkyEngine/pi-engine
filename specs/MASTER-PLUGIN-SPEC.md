To further enhance the Pie in the Sky (PI) 3D Graphics Engine’s PIML (PI Markup Language) and PIScript system, we need to introduce a new `InputPlugin` to handle various input devices, including keyboard, controllers, AR/VR systems, joysticks, flight sticks, controller boards, wheels, pedals, and haptic feedback. This plugin will support user-definable input profiles, broadcast input events via the `Omnibus` event bus, associate controllers with players or NPCs, and enable flexible game profiles for single-player, multiplayer, and cooperative modes (e.g., multiple players controlling a tank’s components). Additionally, the plugin will support up to four-player split-screen gameplay, inspired by Nintendo console multiplayer experiences.

Below, I provide suggestions for the `InputPlugin`, followed by a proposed implementation plan integrated into the existing master specification document. The suggestions address input device support, profile management, event broadcasting, player/controller association, game profiles, and split-screen multiplayer. The updated master specification corrects the package naming (as per the previous correction to `org.piengine.plugin.<module>`) and incorporates the new `InputPlugin` with consistent API definitions, usage examples, and integration guidelines.

---

### Suggestions for InputPlugin

#### 1. Input Device Support
- **Objective**: Support a wide range of input devices, including keyboard, controllers, AR/VR, joysticks, flight sticks, controller boards, wheels, pedals, and haptic feedback.
- **Suggestions**:
  - **Device Abstraction**: Define an `InputDevice` interface to abstract device-specific logic, with implementations for each device type (e.g., `KeyboardDevice`, `VRControllerDevice`).
  - **Device Detection**: Use Java’s `java.awt` for keyboard/mouse and libraries like LWJGL or JInput for controllers, joysticks, and specialized devices.
  - **AR/VR Support**: Integrate with OpenVR/OpenXR for AR/VR headsets and controllers, supporting 6DOF tracking and gestures.
  - **Haptic Feedback**: Provide APIs for vibration/force feedback using device-specific SDKs (e.g., Xbox controller rumble, VR haptic pulses).
  - **Controller Boards**: Support custom boards (e.g., arcade sticks, flight panels) via extensible device drivers.
  - **Cross-Platform**: Ensure compatibility across Windows, macOS, Linux, and WebGL, with fallback to keyboard/mouse for unsupported devices.

#### 2. User-Definable Input Profiles
- **Objective**: Allow users to create profiles mapping controller inputs to in-game functions.
- **Suggestions**:
  - **Profile Format**: Use JSON-based `InputProfile` files (stored in `.pib`) to map inputs (e.g., “button_A”) to actions (e.g., “jump”).
  - **Profile Management**: Provide APIs to create, load, save, and switch profiles dynamically.
  - **Customization UI**: Integrate with `UIPlugin` for in-game profile editing (e.g., button remapping UI).
  - **Default Profiles**: Include prebuilt profiles for common devices (e.g., Xbox controller, keyboard).
  - **Extensibility**: Allow custom mappings for specialized devices (e.g., flight stick throttle).

#### 3. Input Event Broadcasting
- **Objective**: Broadcast input events over the `Omnibus` event bus for processing by other plugins or game logic.
- **Suggestions**:
  - **Event Types**: Define `InputEvent` classes (e.g., `ButtonPressEvent`, `AxisMoveEvent`, `HapticFeedbackEvent`) with device, player, and action details.
  - **Event Dispatch**: Broadcast events via `Omnibus` (e.g., `omnibus.publish(new ButtonPressEvent(playerId, "jump"))`).
  - **Event Filtering**: Allow plugins to subscribe to specific event types or devices (e.g., `UIPlugin` subscribes to menu inputs).
  - **Real-Time Processing**: Ensure low-latency event handling for responsive gameplay.

#### 4. Controller-Player/NPC Association
- **Objective**: Associate input controllers with players or NPCs for precise control.
- **Suggestions**:
  - **Player Mapping**: Link controllers to `PlayerId` via `InputPlugin.assignController(PlayerId, DeviceId)`.
  - **NPC Control**: Allow NPC control via virtual controllers (e.g., AI-driven inputs) or shared player controllers.
  - **Dynamic Reassignment**: Support runtime controller reassignment (e.g., player switching devices).
  - **Multi-Device Support**: Allow one player to use multiple devices (e.g., keyboard + joystick).

#### 5. Game Profiles for Single/Multiplayer Modes
- **Objective**: Support game profiles for single-player, multiplayer, and cooperative modes, including split-screen for up to four players.
- **Suggestions**:
  - **GameProfile Format**: Define JSON-based `GameProfile` files specifying player count, roles, and screen layout (e.g., split-screen quadrants).
  - **Modes**:
    - **Single-Player**: One player with full screen or single controller.
    - **Multiplayer**: Up to four players with split-screen or networked play.
    - **Cooperative**: Multiple players controlling one entity (e.g., tank turret, propulsion, navigation).
  - **Split-Screen**: Use `RasterPlugin` to render multiple viewports (e.g., four quadrants for four players).
  - **Role-Based Control**: Assign roles within cooperative profiles (e.g., `TURRET_OPERATOR`, `DRIVER`) via `AuthorizationPlugin`.
  - **Dynamic Profiles**: Allow runtime profile switching (e.g., single-player to multiplayer).

#### 6. Haptic Feedback
- **Objective**: Provide haptic feedback for immersive gameplay across supported devices.
- **Suggestions**:
  - **Haptic API**: Include methods like `triggerHapticFeedback(DeviceId, HapticConfig)` in `InputPlugin`.
  - **Feedback Types**: Support vibration, force feedback, and directional haptics (e.g., VR controller pulses).
  - **Event-Driven**: Trigger haptics via `Omnibus` events (e.g., `CollisionEvent` triggers rumble).
  - **Customization**: Allow profiles to configure haptic intensity and patterns.

---

### Proposed Implementation Plan

Below is an updated master specification document incorporating the new `InputPlugin` and correcting the package names for all module-specific plugins (sections 3.1–3.19) to use `org.piengine.plugin.<module>`. The document integrates the `InputPlugin` with detailed API definitions, usage examples, and integration guidelines, while preserving all other plugins’ details.


# Master Plugin Specification for PI 3D Graphics Engine (Updated with InputPlugin)

**Author**: Grok 3, built by xAI  
**Date**: June 01, 2025  
**Version**: 1.2 (Added InputPlugin, Corrected Package Names)  
**Current Date and Time**: 11:14 PM EDT, Sunday, June 01, 2025  

## 1. Overview
This master specification document defines the Plugin API for the Pie in the Sky (PI) 3D Graphics Engine’s PIML/PIScript system, consolidating all plugins developed to support features including shaders, animations, physics, particles, clothing/hair/fur, environmental effects, 2D animations, 2D/3D UI, audio, AI-driven tasks, scripting, networking, marketplace transactions, authentication, authorization, storage, and input handling. Each plugin is detailed in a dedicated section, providing API definitions, usage examples, package/module naming, and integration guidelines to enable implementation and usage within the PI engine’s modular architecture.

The specification builds on the core `Plugin` interface, enhanced with plugin signing, version control using `org.piengine.util.Version`, and `pause()`/`unpause()` methods for state management. It ensures cross-platform compatibility (Windows, macOS, Linux, WebGL), extensibility via plugin registration, and seamless integration with the PI editor (`pi-engine-editor`) and asset pipeline (`.pi?` formats: `.pis`, `.pib`, `.pia`, `.pim`, `.pip`).

**Updates**:
- Corrected package names for module-specific plugins to `org.piengine.plugin.<module>` (e.g., `org.piengine.plugin.render`).
- Added `InputPlugin` for handling keyboard, controllers, AR/VR, joysticks, and haptic feedback, with support for user profiles, multiplayer modes, and split-screen gameplay.

## 2. Core Plugin Architecture
### 2.1. Plugin Interface
The core `Plugin` interface defines the minimal contract for all plugins, ensuring consistent lifecycle management, metadata, and interaction patterns.

```java
package org.piengine.core.plugin;

import org.piengine.event.Omnibus;
import org.piengine.util.PluginContext;
import org.piengine.util.PluginMetadata;

public interface Plugin {
    String getPluginId(); // Unique identifier (e.g., "raster:opengl")
    PluginMetadata getMetadata(); // Metadata (version, signature, etc.)
    void initialize(PluginContext context); // Initialize with engine services
    void start(); // Start plugin operations
    void stop(); // Stop plugin operations
    void shutdown(); // Clean up resources
    void registerEventHandlers(Omnibus bus); // Register event listeners
    void handleError(Exception e); // Handle errors
    default void pause() {} // Suspend operations
    default void unpause() {} // Resume operations
}
```

- **PluginMetadata**:
  - `name`: Plugin name (e.g., “RasterPlugin”).
  - `version`: Version string using `org.piengine.util.Version` (e.g., “2.1.0”).
  - `dependencies`: List of required plugins (e.g., `["physics"]`).
  - `platformSupport`: Supported platforms (e.g., `["opengl", "vulkan"]`).
  - `signature`: Base64-encoded digital signature (SHA-256 with RSA).
  - `certificate`: X.509 certificate for signing.
  - `state`: `PluginState` enum (`RUNNING`, `PAUSED`, `STOPPED`).
- **Plugin Signing**: JARs are signed using `java.security`, with signatures in `.sig` files, verified by `MarketplaceVerifier` against a trusted CA.
- **PluginContext**: Provides access to engine services (`SceneManager`, `AssetManager`, `Omnibus`).
- **PluginRegistry**: Manages implementations via `ServiceLoader`, selecting based on metadata (platform, priority).
- **PluginFactory**: Instantiates implementations dynamically (e.g., `RasterPluginFactory.createPlugin("vulkan")`).

### 2.2. Package and Module Naming
- **Core Package**: `org.piengine.core.plugin` (core interfaces and utilities)
- **Module Packages**: `org.piengine.plugin.<module>` (e.g., `org.piengine.plugin.render`)
- **Implementation Packages**: `org.piengine.plugin.<module>.<impl>` (e.g., `org.piengine.plugin.render.opengl`)
- **Modules**:
  - `pi-engine-core`: Core interfaces and utilities.
  - `pi-engine-render`: Rendering plugins.
  - `pi-engine-physics`: Physics plugins.
  - `pi-engine-animation`: Animation plugins.
  - `pi-engine-particles`: Particle plugins.
  - `pi-engine-hairfur`: Clothing/hair/fur plugins.
  - `pi-engine-environment`: Environmental plugins.
  - `pi-engine-2danimation`: 2D animation plugins.
  - `pi-engine-ui`: UI plugins.
  - `pi-engine-audio`: Audio plugins.
  - `pi-engine-ai`: AI plugins.
  - `pi-engine-scripting`: Scripting plugins.
  - `pi-engine-network`: Network plugins.
  - `pi-engine-marketplace`: Marketplace plugins.
  - `pi-engine-auth`: Authentication plugins.
  - `pi-engine-authorization`: Authorization plugins.
  - `pi-engine-storage`: Storage plugins.
  - `pi-engine-input`: Input plugins.

### 2.3. Interaction Patterns
- **Dependency Injection**: `PluginContext` provides engine services.
- **Event-Driven**: Plugins communicate via `Omnibus` events (e.g., `CollisionEvent`).
- **Configuration**: `PluginConfig` interface for JSON-based configs in `.pib`.
- **Thread Safety**: Plugins declare requirements via `isThreadSafe()` in metadata.
- **Versioning**: Uses `org.piengine.util.Version` for compatibility checks.

## 3. Plugin Specifications
Each plugin is detailed with its purpose, API, usage examples, implementations, and integration points.

### 3.1. RasterPlugin
- **Purpose**: Manages rendering of 2D/3D scenes using graphics APIs (OpenGL, Vulkan).
- **Package**: `org.piengine.plugin.render`
- **Module**: `pi-engine-render`
- **API**:
  ```java
  package org.piengine.plugin.render;

  import org.piengine.core.plugin.Plugin;

  public interface RasterPlugin extends Plugin {
      void draw(SceneNode node);
      Shader compileShader(Material material);
      default void drawBatch(List<SceneNode> nodes) { nodes.forEach(this::draw); }
      void registerExtension(String name, Object extension);
  }
  ```
- **Implementations**:
  - `org.piengine.plugin.render.opengl.OpenGLRasterPlugin` (`pi-engine-render-opengl`)
  - `org.piengine.plugin.render.vulkan.VulkanRasterPlugin` (`pi-engine-render-vulkan`)
- **Usage Example**:
  ```piml
  <pi-node id="cube" type="mesh">
    <pi-style material="cube_material" />
  </pi-node>
  ```
  ```piscript
  raster.draw(getNode("cube"));
  ```
- **Integration**:
  - Uses `.pim` for materials.
  - Integrates with `UIPlugin` for HUD rendering and `InputPlugin` for split-screen viewports.
  - Supports `pause()` to halt rendering.
- **Extensibility**: Extensions for custom shaders (e.g., `registerExtension("shader_compiler", ShaderCompiler)`).

### 3.2. AnimationPlugin
- **Purpose**: Manages 2D/3D animations, including keyframes, inverse kinematics, bones, and skinning.
- **Package**: `org.piengine.plugin.animation`
- **Module**: `pi-engine-animation`
- **API**:
  ```java
  package org.piengine.plugin.animation;

  import org.piengine.core.plugin.Plugin;

  public interface AnimationPlugin extends Plugin {
      void processAnimation(SceneNode node, Animation animation);
      void applySkeleton(Skeleton skeleton, SkinningData data);
      void solveIK(IKConstraint constraint);
      void registerAnimationType(String name, AnimationType type);
  }
  ```
- **Implementation**: `org.piengine.plugin.animation.skeletal.SkeletalAnimationPlugin` (`pi-engine-animation-skeletal`)
- **Usage Example**:
  ```piml
  <pi-node id="character" type="skinned_mesh">
    <pi-animation id="walk_cycle" keyframes="walk.pia" />
  </pi-node>
  ```
  ```piscript
  animation.processAnimation(getNode("character"), loadAnimation("walk_cycle"));
  ```
- **Integration**:
  - Uses `.pia` for animations.
  - Links with `AudioPlugin` for lip-sync and `InputPlugin` for animation triggers.
  - Supports `pause()` to freeze animations.
- **Extensibility**: Custom animation types (e.g., `registerAnimationType("procedural", ProceduralAnimation)`).

### 3.3. PhysicsPlugin
- **Purpose**: Simulates rigid/soft body physics, gravity, and collisions using Bullet Physics.
- **Package**: `org.piengine.plugin.physics`
- **Module**: `pi-engine-physics`
- **API**:
  ```java
  package org.piengine.plugin.physics;

  import org.piengine.core.plugin.Plugin;

  public interface PhysicsPlugin extends Plugin {
      void simulate(Scene scene, float dt);
      void addRigidBody(SceneNode node, RigidBodyConfig config);
      void addSoftBody(SceneNode node, SoftBodyConfig config);
      void addConstraint(ConstraintConfig config);
      void applyForce(SceneNode node, Vector3f force);
      void simulateCloth(SceneNode node, ClothConfig config, float dt);
  }
  ```
- **Implementation**: `org.piengine.plugin.physics.bullet.BulletPhysicsPlugin` (`pi-engine-physics-bullet`)
- **Usage Example**:
  ```piml
  <pi-node id="cube" type="mesh">
    <pi-physics type="rigid" mass="1.0" collision-shape="box" />
  </pi-node>
  ```
  ```piscript
  physics.addRigidBody(getNode("cube"), new RigidBodyConfig(1.0, "box"));
  ```
- **Integration**:
  - Uses `.pio` (object physics), `.pib` (scene physics).
  - Interacts with `ParticlePlugin` for collisions, `NetworkPlugin` for state sync, and `InputPlugin` for force inputs.
- **Extensibility**: Custom physics types (e.g., `registerPhysicsType("fluid", FluidSimulation)`).

### 3.4. ParticlePlugin
- **Purpose**: Manages particle systems for effects like sparks, smoke, and fire.
- **Package**: `org.piengine.plugin.particles`
- **Module**: `pi-engine-particles`
- **API**:
  ```java
  package org.piengine.plugin.particles;

  import org.piengine.core.plugin.Plugin;

  public interface ParticlePlugin extends Plugin {
      void simulate(SceneNode node, ParticleSystem system, float dt);
      void render(SceneNode node, ParticleSystem system);
      void addEmitter(SceneNode node, EmitterConfig config);
      void applyBehavior(ParticleSystem system, BehaviorConfig config);
  }
  ```
- **Implementation**: `org.piengine.plugin.particles.gpu.GPUParticlePlugin` (`pi-engine-particles-gpu`)
- **Usage Example**:
  ```piml
  <pi-node id="fire" type="particle">
    <pi-particle-system rate="100" texture="fire_sprite.pit" />
  </pi-node>
  ```
  ```piscript
  particle.addEmitter(getNode("fire"), new EmitterConfig(100));
  ```
- **Integration**:
  - Uses `.pip` for particle systems.
  - Interacts with `PhysicsPlugin` for collisions, `EnvironmentPlugin` for weather effects, and `InputPlugin` for triggered effects.
- **Extensibility**: Custom particle behaviors (e.g., `applyBehavior(system, FlockingBehavior)`).

### 3.5. HairFurPlugin
- **Purpose**: Simulates clothing, hair, and fur with dynamic cloth and strand-based effects.
- **Package**: `org.piengine.plugin.hairfur`
- **Module**: `pi-engine-hairfur`
- **API**:
  ```java
  package org.piengine.plugin.hairfur;

  import org.piengine.core.plugin.Plugin;

  public interface HairFurPlugin extends Plugin {
      void simulateHair(SceneNode node, HairConfig config, float dt);
      void simulateFur(SceneNode node, FurConfig config, float dt);
      void renderHairFur(SceneNode node, HairFurConfig config);
  }
  ```
- **Implementation**: `org.piengine.plugin.hairfur.default.DefaultHairFurPlugin` (`pi-engine-hairfur`)
- **Usage Example**:
  ```piml
  <pi-node id="character" type="skinned_mesh">
    <pi-simulation type="hair" density="1000" curl="0.5" />
  </pi-node>
  ```
  ```piscript
  hairfur.simulateHair(getNode("character"), new HairConfig(1000, 0.5));
  ```
- **Integration**:
  - Uses `.pio` (object simulation), `.pib` (scene simulation).
  - Interacts with `PhysicsPlugin` for cloth, `EnvironmentPlugin` for wind effects, and `InputPlugin` for dynamic adjustments.
- **Extensibility**: Custom simulation types (e.g., `registerSimulationType("braided_hair", BraidedHair)`).

### 3.6. EnvironmentPlugin
- **Purpose**: Manages skybox, clouds, and celestial bodies (sun/moon) for environmental effects.
- **Package**: `org.piengine.plugin.environment`
- **Module**: `pi-engine-environment`
- **API**:
  ```java
  package org.piengine.plugin.environment;

  import org.piengine.core.plugin.Plugin;

  public interface EnvironmentPlugin extends Plugin {
      void simulateSkybox(SceneNode node, SkyboxConfig config, float dt);
      void simulateClouds(SceneNode node, CloudsConfig config, float dt);
      void simulateCelestial(SceneNode node, CelestialConfig config, float dt);
      void renderEnvironment(SceneNode node, EnvironmentConfig config);
  }
  ```
- **Implementation**: `org.piengine.plugin.environment.default.DefaultEnvironmentPlugin` (`pi-engine-environment`)
- **Usage Example**:
  ```piml
  <pi-node id="sky" type="skybox">
    <pi-skybox type="procedural" gradient="sunset" />
  </pi-node>
  ```
  ```piscript
  environment.simulateSkybox(getNode("sky"), new SkyboxConfig("procedural"));
  ```
- **Integration**:
  - Uses `.pis` (scene environment), `.pib` (bundled).
  - Interacts with `WeatherPlugin` for clouds, `ParticlePlugin` for effects, and `InputPlugin` for dynamic environment changes.
- **Extensibility**: Custom environment types (e.g., `registerEnvironmentType("aurora", AuroraSimulation)`).

### 3.7. WeatherPlugin
- **Purpose**: Simulates weather effects like wind, rain, and fog.
- **Package**: `org.piengine.plugin.environment`
- **Module**: `pi-engine-environment`
- **API**:
  ```java
  package org.piengine.plugin.environment;

  import org.piengine.core.plugin.Plugin;

  public interface WeatherPlugin extends Plugin {
      void simulateWeather(Scene scene, WeatherConfig config, float dt);
      void renderWeather(Scene scene, WeatherConfig config);
  }
  ```
- **Implementation**: `org.piengine.plugin.environment.weather.DefaultWeatherPlugin` (`pi-engine-environment`)
- **Usage Example**:
  ```piml
  <pi-weather rain-intensity="0.5" splash="true" />
  ```
  ```piscript
  weather.simulateWeather(getScene(), new WeatherConfig(0.5));
  ```
- **Integration**:
  - Uses `.pis` (scene weather), `.pib` (bundled).
  - Interacts with `ParticlePlugin` for rain, `HairFurPlugin` for wind effects, and `InputPlugin` for weather control.
- **Extensibility**: Custom weather effects (e.g., `registerWeatherEffect("snow", SnowEffect)`).

### 3.8. WaterPlugin
- **Purpose**: Simulates water surfaces (oceans, rivers) with waves and buoyancy.
- **Package**: `org.piengine.plugin.environment`
- **Module**: `pi-engine-environment`
- **API**:
  ```java
  package org.piengine.plugin.environment;

  import org.piengine.core.plugin.Plugin;

  public interface WaterPlugin extends Plugin {
      void simulateWater(SceneNode node, WaterConfig config, float dt);
      void renderWater(SceneNode node, WaterConfig config);
  }
  ```
- **Implementation**: `org.piengine.plugin.environment.water.DefaultWaterPlugin` (`pi-engine-environment`)
- **Usage Example**:
  ```piml
  <pi-node id="ocean" type="water">
    <pi-water wave-height="0.5" buoyancy="1000.0" />
  </pi-node>
  ```
  ```piscript
  water.simulateWater(getNode("ocean"), new WaterConfig(0.5, 1000.0));
  ```
- **Integration**:
  - Uses `.pis` (scene water), `.pib` (bundled).
  - Interacts with `PhysicsPlugin` for buoyancy, `ParticlePlugin` for splashes, and `InputPlugin` for water interactions.
- **Extensibility**: Custom water effects (e.g., `registerWaterEffect("tidal_wave", TidalWaveEffect)`).

### 3.9. TwoDAnimationPlugin
- **Purpose**: Manages 2D animations inspired by Blender’s Grease Pencil, for cartoonish styles and storyboarding.
- **Package**: `org.piengine.plugin.twodanimation`
- **Module**: `pi-engine-2danimation`
- **API**:
  ```java
  package org.piengine.plugin.twodanimation;

  import org.piengine.core.plugin.Plugin;

  public interface TwoDAnimationPlugin extends Plugin {
      void simulateTwoDAnimation(SceneNode node, TwoDAnimationConfig config, float dt);
      void renderTwoDAnimation(SceneNode node, TwoDAnimationConfig config);
      void applyBones(SceneNode node, BoneConfig config);
      void renderStoryboard(StoryboardConfig config);
  }
  ```
- **Implementation**: `org.piengine.plugin.twodanimation.default.DefaultTwoDAnimationPlugin` (`pi-engine-2danimation`)
- **Usage Example**:
  ```piml
  <pi-node id="character_2d" type="two_d">
    <pi-2d-animation stroke="arm_stroke" keyframes="wave.pia" />
  </pi-node>
  ```
  ```piscript
  twodanimation.simulateTwoDAnimation(getNode("character_2d"), loadAnimation("wave"));
  ```
- **Integration**:
  - Uses `.pia` (2D animation), `.pis` (scene-integrated).
  - Interacts with `AudioPlugin` for lip-sync, `UIPlugin` for animated UI, and `InputPlugin` for animation triggers.
- **Extensibility**: Custom animation types (e.g., `registerAnimationType("sketch", SketchAnimation)`).

### 3.10. UIPlugin
- **Purpose**: Manages 2D/3D UI components using JavaFX.
- **Package**: `org.piengine.plugin.ui`
- **Module**: `pi-engine-ui`
- **API**:
  ```java
  package org.piengine.plugin.ui;

  import org.piengine.core.plugin.Plugin;

  public interface UIPlugin extends Plugin {
      void simulateUI(SceneNode node, UIConfig config, float dt);
      void renderUI(SceneNode node, UIConfig config);
      void handleEvent(SceneNode node, UIEvent event);
  }
  ```
- **Implementation**: `org.piengine.plugin.ui.javafx.JavaFXUIPlugin` (`pi-engine-ui-javafx`)
- **Usage Example**:
  ```piml
  <pi-ui-panel id="main_menu" layout="grid">
    <pi-ui-component id="play_button" type="javafx_button" text="Play" />
  </pi-ui-panel>
  ```
  ```piscript
  ui.renderUI(getNode("main_menu"), new UIConfig("grid"));
  ```
- **Integration**:
  - Uses `.pis` (scene UI), `.pib` (bundled).
  - Interacts with `IntegrationPanelPlugin` for legacy/web UI, `AudioPlugin` for UI sounds, and `InputPlugin` for UI input handling.
- **Extensibility**: Custom UI components (e.g., `registerComponent("vr_button", VRButton)`).

### 3.11. IntegrationPanelPlugin
- **Purpose**: Renders legacy (Swing/AWT) and web (HTML/CSS DOM) UI components.
- **Package**: `org.piengine.plugin.ui`
- **Module**: `pi-engine-ui`
- **API**:
  ```java
  package org.piengine.plugin.ui;

  import org.piengine.core.plugin.Plugin;

  public interface IntegrationPanelPlugin extends Plugin {
      void renderJavaFXPanel(SceneNode node, JavaFXConfig config);
      void renderSwingPanel(SceneNode node, SwingConfig config);
      void renderWebView(SceneNode node, WebViewConfig config);
  }
  ```
- **Implementation**: `org.piengine.plugin.ui.integration.DefaultIntegrationPanelPlugin` (`pi-engine-ui-integration`)
- **Usage Example**:
  ```piml
  <pi-integration-panel id="html_form" type="webview" content="form.html" />
  ```
  ```piscript
  integrationpanel.renderWebView(getNode("html_form"), new WebViewConfig("form.html"));
  ```
- **Integration**:
  - Uses `.pis` (scene UI), `.pib` (bundled).
  - Interacts with `UIPlugin` for hybrid UI rendering and `InputPlugin` for input handling.
- **Extensibility**: Custom panel types (e.g., `registerPanelType("custom", CustomPanel)`).

### 3.12. AudioPlugin
- **Purpose**: Manages 2D/3D spatial audio, sound effects, music, and multimedia.
- **Package**: `org.piengine.plugin.audio`
- **Module**: `pi-engine-audio`
- **API**:
  ```java
  package org.piengine.plugin.audio;

  import org.piengine.core.plugin.Plugin;

  public interface AudioPlugin extends Plugin {
      void simulateAudio(SceneNode node, AudioConfig config, float dt);
      void playAudio(SceneNode node, AudioSourceConfig config);
      void applyEffect(SceneNode node, AudioEffectConfig config);
      void manageBus(AudioBusConfig config);
  }
  ```
- **Implementation**: `org.piengine.plugin.audio.openal.OpenALAudioPlugin` (`pi-engine-audio-openal`)
- **Usage Example**:
  ```piml
  <pi-audio-source id="footsteps" type="3d" file="footsteps.wav" />
  ```
  ```piscript
  audio.playAudio(getNode("character"), new AudioSourceConfig("footsteps.wav"));
  ```
- **Integration**:
  - Uses `.pis` (scene audio), `.pib` (bundled).
  - Interacts with `UIPlugin` for UI sounds, `AnimationPlugin` for lip-sync, and `InputPlugin` for sound triggers.
- **Extensibility**: Custom audio effects (e.g., `applyEffect(node, ProceduralNoiseEffect)`).

### 3.13. AIPlugin
- **Purpose**: Supports AI-driven node creation, modification, scenarios, responses, and voice synthesis.
- **Package**: `org.piengine.plugin.ai`
- **Module**: `pi-engine-ai`
- **API**:
  ```java
  package org.piengine.plugin.ai;

  import org.piengine.core.plugin.Plugin;

  public interface AIPlugin extends Plugin {
      void executeAITask(AITaskConfig config);
      void generateNode(AINodeConfig config, SceneNode target);
      void generateResponse(AIResponseConfig config, SceneNode target);
      void synthesizeVoice(AIVoiceConfig config, AudioSource target);
  }
  ```
- **Implementation**: `org.piengine.plugin.ai.default.DefaultAIPlugin` (`pi-engine-ai`)
- **Usage Example**:
  ```piml
  <pi-ai-task id="forest_gen" type="node_creation" prompt="generate a forest" />
  ```
  ```piscript
  ai.generateNode(new AINodeConfig("generate a forest"), getNode("forest_node"));
  ```
- **Integration**:
  - Uses `.pis` (scene AI tasks), `.pib` (bundled).
  - Interacts with `AudioPlugin` for voice synthesis, `NetworkPlugin` for AI-driven state, and `InputPlugin` for AI responses to inputs.
- **Extensibility**: Custom AI tasks (e.g., `registerAITask("procedural_shader", ProceduralShaderTask)`).

### 3.14. ScriptingPlugin
- **Purpose**: Processes PIML/PIScript for scene and script execution.
- **Package**: `org.piengine.plugin.scripting`
- **Module**: `pi-engine-scripting`
- **API**:
  ```java
  package org.piengine.plugin.scripting;

  import org.piengine.core.plugin.Plugin;

  public interface ScriptingPlugin extends Plugin {
      void parsePIML(File pimlFile, SceneNode target);
      void executeScript(String script, Map<String, Object> context);
      void bindScene(SceneNode scene);
      void registerScriptEngine(String name, ScriptEngine engine);
  }
  ```
- **Implementation**: `org.piengine.plugin.scripting.luaj.LuajScriptingPlugin` (`pi-engine-scripting`)
- **Usage Example**:
  ```piml
  <pi-scene version="1.0">
    <pi-node id="cube" type="mesh" />
  </pi-scene>
  ```
  ```piscript
  scripting.executeScript("move(getNode('cube'), vec3(1, 0, 0))", new HashMap<>());
  ```
- **Integration**:
  - Uses `.piml` and `.piscript` files.
  - Interacts with all plugins for scene manipulation, including `InputPlugin` for script-driven input handling.
- **Extensibility**: Custom script engines (e.g., `registerScriptEngine("javascript", JavaScriptEngine)`).

### 3.15. NetworkPlugin
- **Purpose**: Manages client-server communication, game state, and world updates across modes (standalone, LAN, online/server).
- **Package**: `org.piengine.plugin.network`
- **Module**: `pi-engine-network`
- **API**:
  ```java
  package org.piengine.plugin.network;

  import org.piengine.core.plugin.Plugin;

  public interface NetworkPlugin extends Plugin {
      void sendData(GameData data);
      void receiveData(Consumer<GameData> callback);
      GameState saveState(GameState state);
      GameState loadState(String stateId);
      void loadWorld(WorldConfig config);
      void updateWorld(WorldChange change);
  }
  ```
- **Implementations**:
  - `org.piengine.plugin.network.standalone.StandaloneNetworkPlugin` (`pi-engine-network-standalone`)
  - `org.piengine.plugin.network.lan.LANNetworkPlugin` (`pi-engine-network-lan`)
  - `org.piengine.plugin.network.server.ServerNetworkPlugin` (`pi-engine-network-server`)
- **Usage Example**:
  ```piml
  <pi-network mode="standalone" />
  ```
  ```piscript
  network.saveState(new GameState(playerId, inventory));
  ```
- **Integration**:
  - Uses `.pis` (scene network), `.pib` (bundled).
  - Interacts with `StoragePlugin` for state persistence, `AuthenticationPlugin` for player validation, and `InputPlugin` for multiplayer input sync.
- **Extensibility**: Custom network modes (e.g., `registerNetworkMode("cloud", CloudNetwork)`).

### 3.16. MarketplacePlugin
- **Purpose**: Handles in-game transactions and marketplace interactions.
- **Package**: `org.piengine.plugin.marketplace`
- **Module**: `pi-engine-marketplace`
- **API**:
  ```java
  package org.piengine.plugin.marketplace;

  import org.piengine.core.plugin.Plugin;

  public interface MarketplacePlugin extends Plugin {
      TransactionId initiatePurchase(ItemId item, PlayerId player);
      boolean validatePurchase(TransactionId transaction);
      List<Item> listItems(String category);
      void applyPurchase(ItemId item, SceneNode target);
  }
  ```
- **Implementation**: `org.piengine.marketplace.ingame.InGameMarketplacePlugin` (`pi-engine-marketplace`)
- **Usage Example**:
  ```piml
  <pi-ui-panel id="store">
    <pi-ui-component id="buy_health" type="javafx_button" on-click="purchase_health" />
  </pi-ui-panel>
  ```
  ```piscript
  marketplace.initiatePurchase(new ItemId("health_pack"), playerId);
  ```
- **Integration**:
  - Uses `.pis` (scene UI), `.pib` (bundled).
  - Interacts with `AuthenticationPlugin` for player verification, `AuthorizationPlugin` for permissions, and `InputPlugin` for purchase inputs.
- **Extensibility**: Custom store integrations (e.g., `registerStore("steam", SteamStore)`).

### 3.17. AuthenticationPlugin
- **Purpose**: Manages player authentication for online, standalone, and guest modes.
- **Package**: `org.piengine.plugin.auth`
- **Module**: `pi-engine-auth`
- **API**:
  ```java
  package org.piengine.plugin.auth;

  import org.piengine.core.plugin.Plugin;

  public interface AuthenticationPlugin extends Plugin {
      Session login(Credentials credentials);
      Session register(Credentials credentials);
      Session authenticateKey(String keyCode);
      Session createGuestSession();
      void logout(Session session);
      boolean validateSession(Session session);
  }
  ```
- **Implementation**: `org.piengine.plugin.auth.default.DefaultAuthenticationPlugin` (`pi-engine-auth`)
- **Usage Example**:
  ```piml
  <pi-authentication mode="online" />
  ```
  ```piscript
  Session session = authentication.login(new Credentials("user", "pass"));
  ```
- **Integration**:
  - Uses `.pis` (scene auth), `.pib` (bundled).
  - Interacts with `NetworkPlugin` for online mode, `MarketplacePlugin` for premium access, and `InputPlugin` for login inputs.
- **Extensibility**: Custom auth methods (e.g., `registerAuthMethod("biometric", BiometricAuth)`).

### 3.18. AuthorizationPlugin
- **Purpose**: Manages player roles and permissions for access control.
- **Package**: `org.piengine.plugin.authorization`
- **Module**: `pi-engine-authorization`
- **API**:
  ```java
  package org.piengine.plugin.authorization;

  import org.piengine.core.plugin.Plugin;

  public interface AuthorizationPlugin extends Plugin {
      void assignRole(PlayerId player, Role role);
      boolean checkPermission(PlayerId player, Permission permission);
      void restrictAccess(PlayerId player, Action action);
      List<Role> getPlayerRoles(PlayerId player);
  }
  ```
- **Implementation**: `org.piengine.plugin.authorization.default.DefaultAuthorizationPlugin` (`pi-engine-authorization`)
- **Usage Example**:
  ```piml
  <pi-authorization role="PLAYER" player="user123" />
  ```
  ```piscript
  authorization.assignRole(new PlayerId("user123"), Role.PLAYER);
  ```
- **Integration**:
  - Uses `.pis` (scene auth), `.pib` (bundled).
  - Interacts with `AuthenticationPlugin` for role validation, `NetworkPlugin` for multiplayer enforcement, and `InputPlugin` for role-based input restrictions.
- **Extensibility**: Custom role systems (e.g., `registerRoleSystem("custom", CustomRBAC)`).

### 3.19. StoragePlugin
- **Purpose**: Provides client-side and server-side data persistence.
- **Package**: `org.piengine.plugin.storage`
- **Module**: `pi-engine-storage`
- **API**:
  ```java
  package org.piengine.plugin.storage;

  import org.piengine.core.plugin.Plugin;

  public interface StoragePlugin extends Plugin {
      void saveData(String key, Object data);
      Object loadData(String key);
      void deleteData(String key);
      List<Object> queryData(QueryConfig config);
  }
  ```
- **Implementations**:
  - `org.piengine.plugin.storage.inmemory.InMemoryStoragePlugin` (`pi-engine-storage-inmemory`)
  - `org.piengine.plugin.storage.file.FileStoragePlugin` (`pi-engine-storage-file`)
  - `org.piengine.plugin.storage.sqlite.SqliteStoragePlugin` (`pi-engine-storage-sqlite`)
  - `org.piengine.plugin.storage.postgres.PostgresStoragePlugin` (`pi-engine-storage-postgres`)
- **Usage Example**:
  ```piml
  <pi-storage type="inmemory" key="player_state" />
  ```
  ```piscript
  storage.saveData("player_state", new PlayerState(playerId, inventory));
  ```
- **Integration**:
  - Uses `.pis` (scene storage), `.pib` (bundled).
  - Interacts with `NetworkPlugin` for state sync, `AuthenticationPlugin` for secure access, and `InputPlugin` for profile storage.
- **Extensibility**: Custom storage backends (e.g., `registerStorageBackend("mongodb", MongoDBBackend)`).

### 3.20. InputPlugin
- **Purpose**: Manages input devices (keyboard, controllers, AR/VR, joysticks, flight sticks, wheels, pedals) with haptic feedback, user profiles, and multiplayer support.
- **Package**: `org.piengine.plugin.input`
- **Module**: `pi-engine-input`
- **API**:
  ```java
  package org.piengine.plugin.input;

  import org.piengine.core.plugin.Plugin;

  public interface InputPlugin extends Plugin {
      void registerDevice(InputDevice device); // Register input device
      void assignController(PlayerId player, DeviceId device); // Map controller to player/NPC
      void loadProfile(InputProfile profile); // Load user-defined input profile
      void saveProfile(InputProfile profile); // Save input profile
      void triggerHapticFeedback(DeviceId device, HapticConfig config); // Trigger haptic feedback
      void configureGameProfile(GameProfile profile); // Set single/multiplayer mode
      List<InputDevice> getConnectedDevices(); // List active devices
  }
  ```
- **Implementations**:
  - `org.piengine.plugin.input.default.DefaultInputPlugin` (`pi-engine-input`): Supports keyboard, mouse, and standard controllers via LWJGL/JInput.
  - `org.piengine.plugin.input.vr.VRInputPlugin` (`pi-engine-input-vr`): Supports AR/VR via OpenVR/OpenXR.
- **Usage Example**:
  ```piml
  <pi-input profile="default_controller">
    <pi-controller player="user123" device="xbox_controller" />
  </pi-input>
  ```
  ```piscript
  input.loadProfile(new InputProfile("default_controller"));
  input.assignController(new PlayerId("user123"), new DeviceId("xbox_controller"));
  input.triggerHapticFeedback(new DeviceId("xbox_controller"), new HapticConfig(0.5));
  ```
  ```json
  // InputProfile example (in .pib)
  {
    "profileId": "default_controller",
    "mappings": {
      "button_A": "jump",
      "left_stick_x": "move_x",
      "left_stick_y": "move_y"
    },
    "haptic": {
      "intensity": 0.5
    }
  }
  ```
  ```json
  // GameProfile example (in .pib)
  {
    "profileId": "tank_coop",
    "mode": "cooperative",
    "players": [
      {"playerId": "user1", "role": "TURRET_OPERATOR", "viewport": "top_left"},
      {"playerId": "user2", "role": "DRIVER", "viewport": "top_right"}
    ]
  }
  ```
- **Integration**:
  - **Formats**: Uses `.pis` (scene input config), `.pib` (profiles).
  - **Devices**: Supports keyboard/mouse (`java.awt`), controllers/joysticks (`LWJGL/JInput`), AR/VR (`OpenVR/OpenXR`), and custom boards.
  - **Events**: Broadcasts `InputEvent` (e.g., `ButtonPressEvent`) via `Omnibus`, processed by `UIPlugin`, `AnimationPlugin`, or game logic.
  - **Player Association**: Links controllers to `PlayerId` or NPCs via `AuthenticationPlugin`.
  - **Game Profiles**: Configures single-player, multiplayer (up to four players), or cooperative modes with split-screen support via `RasterPlugin`.
  - **Haptic Feedback**: Triggers via `Omnibus` events (e.g., `CollisionEvent`) or direct API calls.
  - **Other Plugins**: Integrates with `NetworkPlugin` for multiplayer input sync, `StoragePlugin` for profile persistence, and `AuthorizationPlugin` for role-based input restrictions.
- **Extensibility**: Custom devices/profiles (e.g., `registerDevice("flight_stick", FlightStickDevice)`).

## 4. Implementation Guidelines
- **Core APIs**: Required for all implementations (e.g., `draw()` in `RasterPlugin`).
- **Extension APIs**: Optional, feature-specific (e.g., `compileShader()` in `RasterPlugin`).
- **Abstract Base Classes**: Provide `Abstract<PluginName>Plugin` for shared logic (e.g., `AbstractRasterPlugin`).
- **Validation Hooks**: Include `validateConfig(PluginConfig)` to ensure compatibility.
- **Documentation**: Mandate Javadoc with usage examples for all APIs.
- **Testing**: Develop a plugin testing framework to validate core and module APIs, including `InputPlugin` for device handling and multiplayer.

## 5. Integration and Usage
- **PI Editor**: Plugins integrate with `pi-engine-editor` via dedicated panels (e.g., “Input Editor” for profile mapping) for real-time previews.
- **Asset Pipeline**: Processes `.piml`, `.picss`, `.piscript` files, caching data in `.pib`.
- **Cross-Platform**: Optimizes for WebGL/low-end devices using media queries and lightweight implementations (e.g., keyboard fallback for `InputPlugin`).
- **Event-Driven**: Uses `Omnibus` for cross-plugin communication (e.g., `InputEvent` triggers animations).
- **Security**: Enforces plugin signing via `MarketplaceVerifier` and encrypts sensitive data (e.g., input profiles).

## 6. Future Enhancements
- **VR/AR Input**: Enhance `InputPlugin` for advanced VR gestures and spatial inputs.
- **Procedural Inputs**: Add AI-driven input simulation for NPCs via `AIPlugin`.
- **Cloud Profiles**: Store input profiles in cloud storage via `StoragePlugin`.
- **Advanced Haptics**: Support complex haptic patterns for immersive devices.



---

### Notes
- **Package Correction**: All module-specific plugins (sections 3.1–3.20) use `org.piengine.plugin.<module>`, with the core `Plugin` interface in `org.piengine.core.plugin`.
- **InputPlugin Integration**: The new `InputPlugin` is fully integrated with existing plugins, supporting multiplayer, split-screen, and cooperative modes.
- **Versioning**: Updated to version 1.2 to reflect the addition of `InputPlugin`.
- **Consistency**: Maintains API structure, usage examples, and integration details from prior specifications.

### Next Steps
- **Feedback Request**: Please review the updated specification, especially the `InputPlugin` (section 3.20), and confirm the package corrections. Indicate if specific plugins or APIs need further detail.
- **Implementation**: Begin coding the `InputPlugin` interface and default implementation, focusing on keyboard/controller support and profile management.
- **Testing**: Develop a testing framework for `InputPlugin`, validating device detection, event broadcasting, and multiplayer modes.
- **Documentation**: Create a “Input Handling Guide” for the PI editor, covering profile creation and split-screen setup.

Let me know how you’d like to proceed or if additional refinements are needed!