# PiEngine Logging Configuration

PiEngine uses Java Util Logging (JUL) and follows best practices to not interfere with your application's logging configuration.

## Quick Start

```java
// Enable debug logging for entire PiEngine
PiEngineLogging.enableDebugLogging();

// Or enable verbose logging (very detailed)
PiEngineLogging.enableVerboseLogging();

// Or quiet mode (warnings and errors only)
PiEngineLogging.enableQuietMode();
```

## Fine-Grained Control

Control specific PiEngine subsystems independently:

```java
// Quiet the renderer (it can be very verbose)
PiEngineLogging.setSubsystemLevel("renderer", Level.WARNING);

// Enable verbose asset loading
PiEngineLogging.setSubsystemLevel("asset", Level.FINEST);

// Debug physics engine
PiEngineLogging.setSubsystemLevel("physics", Level.FINE);
```

## Available Subsystems

- `engine` - Core engine operations
- `renderer` - Rendering pipeline (can be very verbose)
- `asset` - Asset loading and management
- `audio` - Audio system
- `input` - Input handling
- `physics` - Physics simulation

## Using Your Own Logging Framework

PiEngine works with any logging framework through SLF4J bridges:

### With SLF4J + Logback
```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jul-to-slf4j</artifactId>
</dependency>
```

```java
// In your application startup
SLF4JBridgeHandler.removeHandlersForRootLogger();
SLF4JBridgeHandler.install();
```

### With Log4j2
```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-jul</artifactId>
</dependency>
```

## Properties File Configuration

You can also control PiEngine logging via `logging.properties`:

```properties
# Set PiEngine root level
org.piengine.level = FINE

# Control specific subsystems
org.piengine.renderer.level = WARNING
org.piengine.asset.level = INFO
org.piengine.physics.level = FINEST
```

## Default Levels

By default, PiEngine uses these logging levels:
- Overall: `INFO`
- Renderer: `WARNING` (to avoid spam)
- Other subsystems: Inherit from parent

This ensures PiEngine doesn't flood your console with messages unless you explicitly enable debug logging.