# Pie in the Sky (PI) 3D Graphics Engine

**Version**: 1.0  
**Date**: May 31, 2025  
**License**: MIT  

The Pie in the Sky (PI) 3D Graphics Engine is a next-generation, modular 3D graphics engine designed to combine the simplicity of HTML/CSS with the power of modern game engines like Unreal Engine. It targets indie developers, educators, and content creators, offering a flexible, community-driven architecture, a custom editor, and advanced features like AI-driven tools and real-time streaming. The engine is cross-platform, supporting Windows, macOS, Linux, iOS, Android, consoles (PS5, Xbox), VR/AR, and web via WebGL/WASM.

## Overview

The PI engine is built with a modular architecture, allowing developers to extend functionality through plugins while maintaining a lightweight core. The project is organized into several modules:

- **`pi-engine-math` (Java module: `org.piengine.math`)**: Provides mathematical foundations, including 3D coordinates (`Cartesian3D`, `Vector3f`), shapes (`Point3D`), matrices (`Matrix1x3`, `Matrix4x4`), and utility functions (`MathUtils`) for trigonometry, quaternions, and interpolation.
- **`pi-engine-util` (Java module: `org.piengine.util`)**: Offers general-purpose utilities, including high-performance concurrency (`UpgradableReadWriteLock`, `Lockable`), event handling (`Omnibus`), and resource management (`Registration`).
- **`pi-engine-core` (Java module: `org.piengine.core`)**: The main module, providing the core API for scene management (`Scene`, `SceneNode`), plugin system (`Plugin`), apps (`App`), task scheduling (`TaskScheduler` with StructuredTaskScope and Virtual Threads), input handling (`InputManager`), and profiles (`Profile`).
- **Planned Plugin Modules**:
  - `pi-engine-render-opengl` (Java module: `org.piengine.render.opengl`): OpenGL rendering backend.
  - `pi-engine-physics-bullet` (Java module: `org.piengine.physics.bullet`): Bullet Physics integration.
  - `pi-engine-audio-openal` (Java module: `org.piengine.audio.openal`): OpenAL audio support.
  - `pi-engine-ui-imgui` (Java module: `org.piengine.ui.imgui`): ImGui-based UI for the editor.
  - `pi-engine-editor` (Java module: `org.piengine.editor`): Custom editor for scene development and app management.

### Current Deployment State
As of version 1.0, the PI engine includes the core modules (`pi-engine-core`, `pi-engine-math`, `pi-engine-util`) with the following functionality:
- Scene management with `.pis` scenegraphs.
- Plugin system for runtime customization, supporting pausing/unpausing and app-specific initialization.
- App lifecycle management (`App`, `AppManager`) for runtime contexts (e.g., menu, gameplay).
- Multi-threading with StructuredTaskScope and Virtual Threads for efficient task scheduling.
- Basic OpenGL rendering for the MVP.
- Planned plugin modules for advanced rendering, physics, audio, UI, and editor functionality are in development.

## Prerequisites

To use the PI engine, ensure you have the following:

- **Java**: JDK 23 or higher, with `--enable-preview` enabled for features like StructuredTaskScope and Virtual Threads.
- **Maven**: For dependency management and building the project.
- **LWJGL**: Used for rendering, physics, and audio (included as dependencies).
- **Operating System**: Cross-platform support for Windows, macOS, Linux, with planned support for iOS, Android, consoles, VR/AR, and web.

## Installation

The PI engine is available on Maven Central as a set of modular dependencies. To use the core functionality, add the following to your project’s `pom.xml`:

```xml
<dependency>
    <groupId>org.piengine</groupId>
    <artifactId>pi-engine-core</artifactId>
    <version>1.0</version>
</dependency>
```

This will automatically include dependencies on `pi-engine-math` and `pi-engine-util`. For specific features, add the relevant plugin modules once they are available (e.g., `pi-engine-render-opengl`).

### Sample POM for MVP Build
Below is a sample `pom.xml` for an MVP build, integrating the necessary components:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>pi-engine-mvp</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.release>23</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <lwjgl.version>3.3.3</lwjgl.version>
        <snakeyaml.version>2.3</snakeyaml.version>
    </properties>

    <dependencies>
        <!-- Core PI Engine Module -->
        <dependency>
            <groupId>org.piengine</groupId>
            <artifactId>pi-engine-core</artifactId>
            <version>1.0</version>
        </dependency>
        <!-- LWJGL for OpenGL Rendering (MVP) -->
        <dependency>
            <groupId>org.lwjgl</groupId>
            <artifactId>lwjgl</artifactId>
            <version>${lwjgl.version}</version>
        </dependency>
        <dependency>
            <groupId>org.lwjgl</groupId>
            <artifactId>lwjgl-opengl</artifactId>
            <version>${lwjgl.version}</version>
        </dependency>
        <!-- SnakeYAML for Configuration Parsing -->
        <dependency>
            <groupId>org.yaml</groupId>
            <artifactId>snakeyaml</artifactId>
            <version>${snakeyaml.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
                <configuration>
                    <release>${maven.compiler.release}</release>
                    <compilerArgs>
                        <arg>--enable-preview</arg>
                    </compilerArgs>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## Project Structure

The PI engine repository is organized as follows:

```
PiSkyEngine/pisky-engine/
├── pi-engine/                # Parent module
│   ├── pom.xml              # Parent POM
│   └── README.md            # This README
├── pi-engine-math/          # Math module (org.piengine.math)
│   ├── src/main/java/org/piengine/commons/math/
│   ├── pom.xml
│   └── README.md
├── pi-engine-util/          # Util module (org.piengine.util)
│   ├── src/main/java/org/piengine/commons/util/
│   ├── pom.xml
│   └── README.md
├── pi-engine-core/          # Core module (org.piengine.core)
│   ├── src/main/java/org/piengine/core/
│   ├── pom.xml
│   └── README.md
├── pi-engine-render-opengl/ # Planned OpenGL rendering module
├── pi-engine-physics-bullet/ # Planned Bullet Physics module
├── pi-engine-audio-openal/  # Planned OpenAL audio module
├── pi-engine-ui-imgui/      # Planned ImGui UI module
├── pi-engine-editor/        # Planned editor module
└── LICENSE                  # MIT License file
```

## Usage Example

### Creating a Simple Game App with Plugins and Events
This example demonstrates integrating `pi-engine-core`, `pi-engine-math`, and `pi-engine-util` to create a scene, manage an app, handle input via events, and update the scene using a plugin.

```java
import org.piengine.core.scene.Scene;
import org.piengine.core.scene.SceneNode;
import org.piengine.core.plugin.Plugin;
import org.piengine.core.app.App;
import org.piengine.core.app.AppManager;
import org.piengine.math.coordinates.Cartesian3f;
import org.piengine.util.Registration;
import org.piengine.util.eventbus.Omnibus;

// Event for input
class InputEvent {
    private final String action;
    public InputEvent(String action) { this.action = action; }
    public String getAction() { return action; }
}

// Plugin for updating the scene
class UpdatePlugin implements Plugin {
    private final Scene scene;
    private final Omnibus eventBus;
    private Registration inputListener;
    private boolean paused = false;

    public UpdatePlugin(Scene scene, Omnibus eventBus) {
        this.scene = scene;
        this.eventBus = eventBus;
    }

    @Override
    public void init() {
        System.out.println("UpdatePlugin initialized");
        // Register input listener
        inputListener = eventBus.registerListener(InputEvent.class, event -> {
            if (paused) return;
            SceneNode player = scene.getNode("player");
            Cartesian3f position = player.getPosition();
            if ("move_right".equals(event.getAction())) {
                player.setPosition(new Cartesian3f(position.xf() + 1.0f, position.yf(), position.zf()));
                System.out.println("Player moved to: " + player.getPosition().xf());
            }
        });
    }

    @Override
    public void initForApp(App app) {
        System.out.println("UpdatePlugin initialized for app: " + app);
    }

    @Override
    public void update(float dt) {
        if (!paused) {
            System.out.println("UpdatePlugin updating with dt: " + dt);
        }
    }

    @Override
    public void pause() {
        paused = true;
        System.out.println("UpdatePlugin paused");
    }

    @Override
    public void unpause() {
        paused = false;
        System.out.println("UpdatePlugin unpaused");
    }

    @Override
    public void shutdown() {
        System.out.println("UpdatePlugin shutdown");
        inputListener.unregister();
    }

    @Override
    public void processScene(Scene scene) {
        System.out.println("UpdatePlugin processing scene");
    }
}

public class GameExample {
    public static void main(String[] args) throws InterruptedException {
        // Initialize event bus for input handling
        Omnibus eventBus = new OmnibusImpl();

        // Create an app manager
        AppManager appManager = new AppManager();

        // Create a scene with a player node
        Scene scene = new Scene();
        SceneNode playerNode = scene.createNode("player");
        playerNode.setPosition(new Cartesian3f(0.0f, 0.0f, 0.0f));

        // Define a game app with an update plugin
        App gameApp = new App("GameApp") {
            private Scene appScene;
            private AppStatus status = AppStatus.UNINITIALIZED;

            @Override
            public void initialize() {
                appScene = scene;
                status = AppStatus.INITIALIZED;
                System.out.println("GameApp initialized");
            }

            @Override
            public void start() {
                status = AppStatus.RUNNING;
                System.out.println("GameApp started");
            }

            @Override
            public void update(float deltaTime) {
                System.out.println("GameApp updating");
            }

            @Override
            public void pause() { status = AppStatus.PAUSED; }
            @Override
            public void unpause() { status = AppStatus.RUNNING; }
            @Override
            public void stop() { status = AppStatus.STOPPED; }
            @Override
            public void cleanup() { status = AppStatus.TERMINATED; }
            @Override
            public Scene getScene() { return appScene; }
            @Override
            public AppStatus getStatus() { return status; }
            @Override
            public void addPlugin(Plugin plugin) { plugin.initForApp(this); }
            @Override
            public void removePlugin(Plugin plugin) { plugin.shutdown(); }
        };
        gameApp.addPlugin(new UpdatePlugin(scene, eventBus));
        appManager.addApp(gameApp);

        // Switch to the game app
        appManager.setCurrentApp("GameApp");

        // Simulate input event (e.g., player moves right)
        eventBus.publish(new InputEvent("move_right"));

        // Simulate an update loop
        appManager.update(0.016f); // 60 FPS frame time

        // Pause and update again
        gameApp.pause();
        appManager.update(0.016f);

        // Unpause and update
        gameApp.unpause();
        appManager.update(0.016f);

        // Shutdown the app manager
        appManager.shutdown();
    }
}
```

**Output**:
```
UpdatePlugin initialized for app: GameApp@...
GameApp initialized
GameApp started
UpdatePlugin initialized
Player moved to: 1.0
GameApp updating
UpdatePlugin updating with dt: 0.016
UpdatePlugin paused
GameApp updating
UpdatePlugin unpaused
GameApp updating
UpdatePlugin updating with dt: 0.016
UpdatePlugin shutdown
```

## Plugin Development Guide

To create a custom plugin, implement the `Plugin` interface:

1. **Define the Plugin Class**:
   ```java
   import org.piengine.core.plugin.Plugin;
   import org.piengine.core.scene.Scene;
   import org.piengine.core.app.App;

   public class CustomPlugin implements Plugin {
       @Override
       public void init() { System.out.println("CustomPlugin initialized"); }

       @Override
       public void initForApp(App app) { System.out.println("CustomPlugin initialized for app: " + app); }

       @Override
       public void update(float dt) { System.out.println("CustomPlugin updating with dt: " + dt); }

       @Override
       public void pause() { System.out.println("CustomPlugin paused"); }

       @Override
       public void unpause() { System.out.println("CustomPlugin unpaused"); }

       @Override
       public void shutdown() { System.out.println("CustomPlugin shutdown"); }

       @Override
       public void processScene(Scene scene) { System.out.println("CustomPlugin processing scene"); }
   }
   ```

2. **Package as a `.pip` File**:
   - Create a JAR file containing your plugin class.
   - ZIP the JAR into a `.pip` file (e.g., `custom-plugin.pip`).

3. **Load the Plugin**:
   ```yaml
   apps:
     - id: GameApp
       scene: game.pis
       plugins: [custom-plugin.pip]
   ```

4. **Register with ServiceLoader** (Optional):
   - Add a `META-INF/services/org.piengine.core.plugin.Plugin` file in your JAR, listing your plugin class (e.g., `com.example.CustomPlugin`).

## Troubleshooting

- **Java Version Issues**: Ensure you’re using JDK 23+ with `--enable-preview`. Add the following to your JVM arguments:
  ```
  --enable-preview
  ```
- **LWJGL Dependencies**: If you encounter native library errors, ensure the correct LWJGL natives are included for your platform. Add the appropriate dependency:
  ```xml
  <dependency>
      <groupId>org.lwjgl</groupId>
      <artifactId>lwjgl</artifactId>
      <version>3.3.3</version>
      <classifier>${os.detected.classifier}</classifier>
  </dependency>
  ```
- **Plugin Not Loading**: Verify the `.pip` file is a valid ZIP containing a JAR with a `META-INF/services/org.piengine.core.plugin.Plugin` entry.
- **Scene Not Rendering**: For the MVP, ensure basic OpenGL rendering is set up. Advanced rendering backends (e.g., Vulkan) are planned for future releases.

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository: `https://github.com/PiSkyEngine/pisky-engine`.
2. Create a branch for your feature (`git checkout -b feature/new-feature`).
3. Commit your changes (`git commit -m "Add new feature"`).
4. Push to your branch (`git push origin feature/new-feature`).
5. Open a pull request on GitHub.

For more details, see the [Contributing Guide](https://github.com/PiSkyEngine/pisky-engine/wiki/Contributing) in the project wiki.

## Links
- **Project Website**: [www.piengine.org](https://www.piengine.org)
- **GitHub Wiki**: [PiSkyEngine/pisky-engine Wiki](https://github.com/PiSkyEngine/pisky-engine/wiki)
- **Repository**: [PiSkyEngine/pisky-engine](https://github.com/PiSkyEngine/pisky-engine)
- **Issue Tracker**: [GitHub Issues](https://github.com/PiSkyEngine/pisky-engine/issues)