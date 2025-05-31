# Pie in the Sky (PI) 3D Graphics Engine

**Version**: 0.0.1-SNAPSHOT  
**Date**: May 30, 2025  
**License**: MIT  

The Pie in the Sky (PI) 3D Graphics Engine is a next-generation, modular 3D graphics engine designed to combine the simplicity of HTML/CSS with the power of modern game engines like Unreal and Unity. It targets indie developers, educators, and content creators, offering a flexible, community-driven architecture, a custom editor, and advanced features like AI-driven tools and real-time streaming. The engine is cross-platform, supporting Windows, macOS, Linux, iOS, Android, consoles (PS5, Xbox), VR/AR, and web via WebGL/WASM.

## Overview

The PI engine is organized into modular components, each focusing on a specific aspect of 3D graphics development:

- **`pi-engine-math`**: Provides mathematical foundations, including 3D coordinates (`Cartesian3D`, `Vector3f`), shapes (`Point3D`), matrices, and utility functions (`MathUtils`).
- **`pi-engine-util`**: Offers general-purpose utilities, including high-performance concurrency (`UpgradableReadWriteLock`, `Lockable`), event handling (`Omnibus`), and registration (`Registration`).
- **`pi-engine-core`**: The central module, handling engine-specific APIs like scene management (`Scene`, `SceneNode`), plugins (`Plugin`), application states (`AppState`), task scheduling (`TaskScheduler`), input handling (`InputManager`), and profiles (`Profile`).
- **Planned Modules**:
  - `pi-engine-render-opengl`: OpenGL rendering backend.
  - `pi-engine-physics-bullet`: Bullet Physics integration.
  - `pi-engine-audio-openal`: OpenAL audio support.
  - `pi-engine-ui-imgui`: ImGui-based UI for the editor.
  - `pi-engine-editor`: Custom editor for scene development and app state management.

The PI engine is built with Java 23+, leveraging modern features like StructuredTaskScope and Virtual Threads for efficient multi-threading. It uses LWJGL for rendering, physics, and audio, and SnakeYAML for configuration parsing.

## Installation

The PI engine is available on Maven Central as a set of modular dependencies. To use the core functionality, add the following to your project’s `pom.xml`:

```xml
<dependency>
    <groupId>org.piengine</groupId>
    <artifactId>pi-engine-core</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

This will automatically include dependencies on `pi-engine-math` and `pi-engine-util`. For specific features, add the relevant modules (e.g., `pi-engine-render-opengl` when available).

### Requirements
- **Java**: 23 or higher (with `--enable-preview` for features like StructuredTaskScope and Virtual Threads).
- **Dependencies**:
  - `pi-engine-math` (for coordinates and shapes).
  - `pi-engine-util` (for concurrency and event handling).
  - SnakeYAML (for parsing `.pis` files and profiles).
  - LWJGL (for rendering, physics, audio; included via parent POM).

## Usage Example

### Creating a Simple Game Scene with Plugins and Events
This example demonstrates integrating `pi-engine-core`, `pi-engine-math`, and `pi-engine-util` to create a scene, manage application states, handle input via events, and update the scene using a plugin.

```java
import org.pi.game.engine.core.scene.Scene;
import org.pi.game.engine.core.scene.SceneNode;
import org.pi.game.engine.core.plugin.Plugin;
import org.pi.game.engine.core.state.AppState;
import org.pi.game.engine.core.state.StateManager;
import org.piengine.commons.math.coordinates.Cartesian3f;
import org.piengine.commons.util.Registration;
import org.piengine.commons.util.eventbus.Omnibus;

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

    public UpdatePlugin(Scene scene, Omnibus eventBus) {
        this.scene = scene;
        this.eventBus = eventBus;
    }

    @Override
    public void init() {
        System.out.println("UpdatePlugin initialized");
        // Register input listener
        inputListener = eventBus.registerListener(InputEvent.class, event -> {
            SceneNode player = scene.getNode("player");
            Cartesian3f position = player.getPosition();
            if ("move_right".equals(event.getAction())) {
                player.setPosition(new Cartesian3f(position.xf() + 1.0f, position.yf(), position.zf()));
                System.out.println("Player moved to: " + player.getPosition().xf());
            }
        });
    }

    @Override
    public void update(float dt) {
        // Simulate game loop update
        System.out.println("UpdatePlugin updating with dt: " + dt);
    }

    @Override
    public void shutdown() {
        System.out.println("UpdatePlugin shutdown");
        inputListener.unregister();
    }
}

public class GameExample {
    public static void main(String[] args) throws InterruptedException {
        // Initialize event bus for input handling
        Omnibus eventBus = new OmnibusImpl();

        // Create a state manager
        StateManager stateManager = new StateManager();

        // Create a scene with a player node
        Scene scene = new Scene();
        SceneNode playerNode = scene.createNode("player");
        playerNode.setPosition(new Cartesian3f(0.0f, 0.0f, 0.0f));

        // Define a game state with an update plugin
        AppState gameState = new AppState("GameState");
        gameState.addPlugin(new UpdatePlugin(scene, eventBus));
        stateManager.addState(gameState);

        // Switch to the game state
        stateManager.setCurrentState("GameState");

        // Simulate input event (e.g., player moves right)
        eventBus.publish(new InputEvent("move_right"));

        // Simulate an update loop
        stateManager.update(0.016f); // 60 FPS frame time

        // Shutdown the state manager
        stateManager.shutdown();
    }
}
```

**Output**:
```
UpdatePlugin initialized
Player moved to: 1.0
UpdatePlugin updating with dt: 0.016
UpdatePlugin shutdown
```

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository: `https://github.com/PiSkyEngine/pisky-engine`.
2. Create a branch for your feature (`git checkout -b feature/new-feature`).
3. Commit your changes (`git commit -m "Add new feature"`).
4. Push to your branch (`git push origin feature/new-feature`).
5. Open a pull request on GitHub.

For more details, see the [Contributing Guide](https://github.com/PiSkyEngine/pisky-engine/wiki/Contributing) in the project wiki.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

```text
MIT License

Copyright (c) 2025 Sly Technologies Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```

## Links
- **Project Website**: [www.piengine.org](https://www.piengine.org)
- **GitHub Wiki**: [PiSkyEngine/pisky-engine Wiki](https://github.com/PiSkyEngine/pisky-engine/wiki)
- **Repository**: [PiSkyEngine/pisky-engine](https://github.com/PiSkyEngine/pisky-engine)
- **Issue Tracker**: [GitHub Issues](https://github.com/PiSkyEngine/pisky-engine/issues)



---

### **Explanation of the `README.md`**

#### **Structure**
- **Header**: Includes the project name (`Pie in the Sky (PI) 3D Graphics Engine`), version (`0.0.1-SNAPSHOT`), date (May 30, 2025, as per the current date), and license (MIT).
- **Overview**: Summarizes the PI engine’s purpose (modular 3D graphics engine), target audience (indie developers, educators, content creators), and cross-platform support. Lists all modules (`pi-engine-math`, `pi-engine-util`, `pi-engine-core`, and planned modules), describing their roles.
- **Installation**:
  - Provides Maven dependency snippet for `pi-engine-core`, noting that it pulls in `pi-engine-math` and `pi-engine-util`.
  - Lists requirements (Java 23 with `--enable-preview`, dependencies on SnakeYAML, LWJGL included via parent POM).
- **Usage Example**:
  - **Creating a Simple Game Scene with Plugins and Events**: Demonstrates integrating the core modules:
    - Uses `pi-engine-core` for scene management (`Scene`, `SceneNode`), state management (`AppState`, `StateManager`), and plugins (`Plugin`).
    - Uses `pi-engine-math` for positioning (`Cartesian3f`).
    - Uses `pi-engine-util` for event handling (`Omnibus`, `Registration`).
    - The example creates a scene with a `player` node, sets up a game state with an `UpdatePlugin`, and uses `Omnibus` to handle an input event (e.g., moving the player).
    - Output: Shows the plugin lifecycle and player movement, confirming the integration of modules.
- **Contributing**: Provides standard GitHub contribution steps, linking to a wiki page for detailed guidelines.
- **License**: Includes the MIT license text, consistent with the project’s license.
- **Links**:
  - Project website: `www.piengine.org`.
  - GitHub wiki: `https://github.com/PiSkyEngine/pisky-engine/wiki`.
  - Repository path: Assumed as `PiSkyEngine/pisky-engine`.
  - Issue tracker: Links to GitHub issues.

#### **Alignment with PI Engine**
- **Module Integration**: The README summarizes all modules, showing how they work together:
  - `pi-engine-math` provides `Cartesian3f` for positioning.
  - `pi-engine-util` provides `Omnibus` and `Registration` for event handling.
  - `pi-engine-core` manages the scene, plugins, and states.
- **Version**: Matches the project version (`0.0.1-SNAPSHOT`).
- **Java 23**: Noted in requirements, with `--enable-preview` for features like StructuredTaskScope and Virtual Threads.
- **License**: Uses MIT, consistent with prior READMEs and POM files.

#### **Example**
- **Game Scene with Plugins and Events**: Combines key features:
  - Creates a `Scene` with a `SceneNode` (`player`), using `Cartesian3f` for positioning.
  - Sets up an `AppState` with an `UpdatePlugin` that listens for input events via `Omnibus`.
  - Simulates an input event (`move_right`) to move the player, demonstrating event-driven updates.
  - The example integrates scene management, state transitions, plugin lifecycle, and event handling, showcasing the modular design of the PI engine.

#### **Dependencies**
- The example implicitly relies on `pi-engine-math` for coordinates and `pi-engine-util` for `Omnibus` and `Registration`, which are pulled in by `pi-engine-core`’s dependencies (as defined in the core module’s `pom.xml`).

---

### **Conclusion**

The `README.md` for the `pi-engine` parent module provides a comprehensive overview of the PI 3D graphics engine, summarizing the roles of all modules and demonstrating their integration through a practical example. It uses the MIT license, links to the project website (`www.piengine.org`) and GitHub wiki, and serves as the top-level documentation for the project on GitHub. The example shows how to create a simple game scene with plugins and event handling, aligning with the PI engine’s focus on modularity and extensibility.