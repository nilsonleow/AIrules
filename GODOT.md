# Godot GDScript Agent Rules

## Project Context
You are developing a game with Godot Engine 4.x using GDScript. The project uses node-based scene composition, signal-driven communication, and Godot's built-in physics, animation, and UI systems.

## Code Style
- Use `snake_case` for variables, functions, and file names: `player_speed`, `on_body_entered`, `player.gd`.
- Use `PascalCase` for class names and node types: `class PlayerController`, `StateMachine`.
- Use `UPPER_SNAKE_CASE` for constants: `const MAX_SPEED: float = 500.0`.
- Prefix private members with underscore: `var _health: int`, `func _update_ui() -> void`.
- Use static typing everywhere: `var speed: float = 5.0`, `func move(delta: float) -> void:`.
- Add `##` doc comments on exported variables and public functions.
- Keep scripts under 200 lines — extract large scripts into focused components or autoloads.
- Use `@onready var sprite: Sprite2D = $Sprite2D` instead of setting node references in `_ready()`.

## Project Structure
- Organize by feature: `scenes/player/`, `scenes/enemies/`, `scenes/ui/`, `scenes/world/`.
- Co-locate each script with its `.tscn` file: `scenes/player/player.tscn` and `scenes/player/player.gd`.
- Store shared resources (themes, curves, audio streams) in `resources/` with subdirectories by type.
- Place autoload singletons in `autoloads/`: `autoloads/game_state.gd`, `autoloads/audio_manager.gd`.
- Register autoloads in Project Settings → Autoload, not via code.
- Store raw assets in `assets/` with subdirectories: `assets/textures/`, `assets/audio/`, `assets/fonts/`.

## Node Architecture & Signals
- Design every scene as a self-contained unit with clear input (exported variables) and output (signals).
- Use signals for child-to-parent communication — never call methods on a parent node directly.
- Define custom signals at the top of the script with typed parameters: `signal health_changed(new_value: int)`.
- Connect signals in code using `signal_name.connect(callback)` rather than the editor for logic-critical connections.
- Use autoload singletons only for truly global systems: audio manager, save system, event bus.
- Prefer composition — attach behavior scripts to child nodes rather than building deep inheritance hierarchies.

## Export Variables & Inspector
- Use `@export` to expose configurable parameters in the Inspector: `@export var speed: float = 200.0`.
- Use `@export_range` for clamped numeric values: `@export_range(0.0, 1.0, 0.05) var friction: float = 0.8`.
- Use `@export_group("Movement")` to organize Inspector properties into collapsible groups.
- Use `@export var scene: PackedScene` for spawnable objects to keep references editor-configurable.

## State Machines
- Implement FSMs for all entities with complex behavior: players, enemies, interactive objects.
- Create a base `State` class extending `Node` with virtual methods: `enter()`, `exit()`, `update(delta)`, `physics_update(delta)`.
- Mount state nodes as children of a `StateMachine` node on the entity.
- Transition states through the state machine: `state_machine.transition_to("Run")`, never directly.
- Log state transitions during development: `print("[SM] %s → %s" % [current_state.name, next_state.name])`.

## Physics & Input
- Define all input actions in Project Settings → Input Map — never hardcode `KEY_W` or `MOUSE_BUTTON_LEFT` in scripts.
- Handle gameplay input in `_unhandled_input()` — it only fires if no Control node consumed the event.
- Use `CharacterBody2D.move_and_slide()` for player movement; set `up_direction = Vector2.UP` for platformers.
- Set collision layers and masks intentionally: layer 1 = player, layer 2 = enemies, layer 3 = projectiles.
- Use `Area2D` for triggers, pickups, and detection zones — `collision_entered` not physics response.
- Apply forces to `RigidBody2D/3D` via `apply_impulse()` or `apply_force()` rather than setting velocity directly.

## Performance
- Profile with Godot's built-in profiler (Debugger → Profiler) — focus on both CPU frame time and draw calls.
- Use object pooling for frequently spawned objects: bullets, particles, enemy spawns.
- Preload scenes and resources at startup: `const BULLET_SCENE = preload("res://scenes/projectile/bullet.tscn")`.
- Disable `_process` and `_physics_process` with `set_process(false)` when nodes are idle to reduce per-frame overhead.
- Use `PackedByteArray`, `PackedVector2Array`, and other typed packed arrays for bulk data operations.
- Avoid creating dictionaries or arrays inside `_process` — allocate them at ready time and reuse.

## Testing & Export
- Use GUT (Godot Unit Testing) for unit tests on pure GDScript logic: parsers, state machines, utilities.
- Export for testing early — editor behavior can differ from exported builds, especially on mobile.
- Configure export presets in the Export dialog for each target platform before reaching milestone builds.
- Use `--headless --script test_runner.gd` in CI for automated test runs.
