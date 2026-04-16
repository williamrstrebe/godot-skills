\---

name: godot

description: >

&#x20; Use this skill whenever the user wants to create, scaffold, architect, or improve a Godot Engine game project.

&#x20; Triggers include: starting a new Godot game, structuring a Godot project, writing GDScript or C# for Godot,

&#x20; asking about Godot best practices, scene/node architecture, signals, autoloads, resource management, input handling,

&#x20; state machines, UI in Godot, game loops, save/load systems, or anything related to Godot 4.x development.

&#x20; Use this even when the user doesn't say "Godot skill" — if they mention GDScript, scenes, nodes, .tscn files, or

&#x20; @export variables, this skill is relevant. Applies to both 2D and 3D; consult the appropriate reference file for

&#x20; dimension-specific guidance after reading this main skill.

\---



\# Godot Game Project Skill



This skill guides the creation and architecture of Godot 4.x game projects with professional standards:

clean code, reusability, readability, and maintainability. It is \*\*dimension-agnostic\*\* by default — read

`references/godot-2d.md` or `references/godot-3d.md` for additional guidance when the project type is known.



\---



\## 1. Project Structure



A well-organized Godot project separates concerns clearly. Use this canonical layout:



```

project/

├── project.godot

├── icon.svg

├── assets/

│   ├── audio/

│   │   ├── music/

│   │   └── sfx/

│   ├── fonts/

│   ├── sprites/          # 2D only

│   ├── models/           # 3D only

│   ├── textures/

│   └── shaders/

├── scenes/

│   ├── ui/

│   ├── levels/

│   ├── entities/

│   │   ├── player/

│   │   └── enemies/

│   └── components/       # Reusable sub-scenes

├── scripts/

│   ├── autoloads/        # Singletons / global state

│   ├── resources/        # Custom Resource types

│   ├── components/       # Reusable script logic

│   └── utils/            # Pure utility functions

├── data/

│   └── \*.tres / \*.res    # Resource data files

└── tests/                # GUT or gdUnit4 tests

```



\*\*Rules:\*\*

\- Every major entity (Player, Enemy, Item) lives in its own subfolder containing both `.tscn` and `.gd` files.

\- Assets are never placed in `scripts/` and scripts are never placed in `assets/`.

\- Scene files (`.tscn`) and their primary scripts (`.gd`) share the same name and live together.



\---



\## 2. Node \& Scene Architecture



\### Scene Composition over Inheritance

Prefer \*\*composing\*\* scenes from smaller sub-scenes rather than deep inheritance chains.



```

PlayerScene

├── CharacterBody2D / CharacterBody3D  ← Root (player.gd)

├── CollisionShape2D / CollisionShape3D

├── Sprite2D / MeshInstance3D

├── AnimationPlayer

├── components/

│   ├── HealthComponent.tscn           ← Reusable sub-scene

│   ├── HurtboxComponent.tscn

│   └── StateMachineComponent.tscn

└── ui/

&#x20;   └── HealthBar.tscn

```



\### Node Naming Conventions

\- \*\*PascalCase\*\* for node names: `PlayerSprite`, `MainCamera`, `HurtboxArea`

\- \*\*snake\_case\*\* for variables, functions, files: `player.gd`, `health\_component.gd`

\- \*\*ALL\_CAPS\*\* for constants: `MAX\_SPEED`, `GRAVITY`

\- \*\*\_underscore prefix\*\* for private members: `\_current\_state`, `\_is\_grounded`



\### Typed GDScript — Always Use Types

```gdscript

\# Bad

var speed = 200

func move(delta):

&#x20;   position.x += speed \* delta



\# Good

var speed: float = 200.0

func move(delta: float) -> void:

&#x20;   position.x += speed \* delta

```



Type annotations improve autocomplete, catch bugs early, and document intent.



\---



\## 3. GDScript Standards



\### Script Template

Every script should begin with a clear header block:



```gdscript

class\_name PlayerController

extends CharacterBody2D



\## Controls player movement, input handling, and state transitions.

\## Emits \[signal died] when health reaches zero.

\## Depends on: HealthComponent, StateMachine



\# ── Signals ────────────────────────────────────────────────────────

signal died

signal jumped



\# ── Constants ──────────────────────────────────────────────────────

const COYOTE\_TIME: float = 0.1

const JUMP\_BUFFER\_TIME: float = 0.12



\# ── Exports ────────────────────────────────────────────────────────

@export var move\_speed: float = 300.0

@export var jump\_force: float = 600.0

@export var health\_component: HealthComponent



\# ── Private Variables ──────────────────────────────────────────────

var \_velocity: Vector2 = Vector2.ZERO

var \_coyote\_timer: float = 0.0



\# ── Onready ────────────────────────────────────────────────────────

@onready var \_animation\_player: AnimationPlayer = $AnimationPlayer

@onready var \_sprite: Sprite2D = $Sprite2D



\# ── Lifecycle ──────────────────────────────────────────────────────

func \_ready() -> void:

&#x20;   health\_component.died.connect(\_on\_died)



func \_physics\_process(delta: float) -> void:

&#x20;   \_handle\_movement(delta)



\# ── Public Methods ─────────────────────────────────────────────────

func take\_damage(amount: float) -> void:

&#x20;   health\_component.take\_damage(amount)



\# ── Private Methods ────────────────────────────────────────────────

func \_handle\_movement(delta: float) -> void:

&#x20;   pass



\# ── Signal Callbacks ───────────────────────────────────────────────

func \_on\_died() -> void:

&#x20;   died.emit()

&#x20;   queue\_free()

```



\### Section ordering (always follow this):

1\. `class\_name` + `extends`

2\. `## docstring`

3\. Signals

4\. Constants

5\. `@export` vars

6\. Regular vars (private with `\_`)

7\. `@onready` vars

8\. `\_ready()`, `\_process()`, `\_physics\_process()`

9\. Public methods

10\. Private methods (`\_`)

11\. Signal callbacks (`\_on\_\*`)



\---



\## 4. Signals — The Godot Way



Signals decouple systems. \*\*Emit upward, call downward\*\* — child nodes tell parents things happened; parents call methods on children directly.



```gdscript

\# ✅ Child emits, parent listens

\# enemy.gd

signal died(enemy: Enemy)

func \_on\_health\_depleted() -> void:

&#x20;   died.emit(self)



\# level.gd

func \_ready() -> void:

&#x20;   enemy.died.connect(\_on\_enemy\_died)



func \_on\_enemy\_died(enemy: Enemy) -> void:

&#x20;   enemy\_count -= 1

```



```gdscript

\# ❌ Avoid — child calling parent directly (tight coupling)

func \_on\_health\_depleted() -> void:

&#x20;   get\_parent().remove\_enemy(self)

```



\*\*Signal naming:\*\* use past tense for events that already happened (`died`, `jumped`, `item\_collected`), and descriptive names for requests (`attack\_requested`).



\---



\## 5. Autoloads (Singletons)



Use Autoloads \*\*sparingly\*\* — only for truly global systems. Register them in `Project > Project Settings > Autoload`.



| Autoload | Purpose |

|---|---|

| `GameManager` | Game state, scene transitions, pause |

| `AudioManager` | Pooled audio, music layers |

| `SaveManager` | Save/load game data |

| `EventBus` | Global signal relay (optional) |

| `InputManager` | Input remapping, gamepad detection |



```gdscript

\# autoloads/game\_manager.gd

class\_name GameManager

extends Node



signal game\_paused(is\_paused: bool)



var current\_level: String = ""

var score: int = 0



func change\_scene(path: String) -> void:

&#x20;   get\_tree().change\_scene\_to\_file(path)



func pause(value: bool) -> void:

&#x20;   get\_tree().paused = value

&#x20;   game\_paused.emit(value)

```



\*\*EventBus pattern\*\* (optional but powerful for decoupled global events):

```gdscript

\# autoloads/event\_bus.gd

extends Node

signal player\_died

signal level\_completed(level\_id: int)

signal item\_picked\_up(item\_type: StringName)

```



\---



\## 6. Custom Resources



Use `Resource` subclasses for data-driven design — stats, item definitions, ability configs, level data.



```gdscript

\# scripts/resources/character\_stats.gd

class\_name CharacterStats

extends Resource



@export var max\_health: float = 100.0

@export var move\_speed: float = 300.0

@export var jump\_force: float = 600.0

@export var attack\_damage: float = 25.0

@export var attack\_cooldown: float = 0.5

```



Use `.tres` files to create instances without code. This enables:

\- Designers to edit values without touching GDScript

\- Easy A/B testing of different stat sets

\- Sharing resource instances between scenes (flyweight pattern)



```gdscript

\# In player.gd

@export var stats: CharacterStats



func \_ready() -> void:

&#x20;   \_health = stats.max\_health

&#x20;   \_speed = stats.move\_speed

```



\---



\## 7. State Machines



Always use a state machine for entities with complex behavior. Avoid long `if/elif` chains in `\_process()`.



```gdscript

\# scripts/components/state\_machine.gd

class\_name StateMachine

extends Node



@export var initial\_state: State



var current\_state: State



func \_ready() -> void:

&#x20;   for child in get\_children():

&#x20;       if child is State:

&#x20;           child.state\_machine = self

&#x20;   transition\_to(initial\_state)



func \_process(delta: float) -> void:

&#x20;   current\_state.update(delta)



func \_physics\_process(delta: float) -> void:

&#x20;   current\_state.physics\_update(delta)



func transition\_to(new\_state: State) -> void:

&#x20;   if current\_state:

&#x20;       current\_state.exit()

&#x20;   current\_state = new\_state

&#x20;   current\_state.enter()

```



```gdscript

\# scripts/components/state.gd

class\_name State

extends Node



var state\_machine: StateMachine



func enter() -> void: pass

func exit() -> void: pass

func update(\_delta: float) -> void: pass

func physics\_update(\_delta: float) -> void: pass

```



States live as child nodes of `StateMachine`. Each state is its own script/scene: `IdleState`, `RunState`, `JumpState`, `AttackState`.



\---



\## 8. Input Handling



Always use \*\*Input Map\*\* actions (not hardcoded keys). Define actions in `Project > Project Settings > Input Map`.



```gdscript

\# ✅ Good — uses Input Map, remappable

func \_get\_movement\_input() -> Vector2:

&#x20;   return Input.get\_vector("move\_left", "move\_right", "move\_up", "move\_down")



func \_physics\_process(delta: float) -> void:

&#x20;   var direction := \_get\_movement\_input()

&#x20;   if Input.is\_action\_just\_pressed("jump"):

&#x20;       \_try\_jump()

```



```gdscript

\# ❌ Avoid — hardcoded, not remappable

if Input.is\_key\_pressed(KEY\_SPACE):

&#x20;   jump()

```



Standard action names to define: `move\_left`, `move\_right`, `move\_up`, `move\_down`, `jump`, `attack`, `interact`, `pause`, `ui\_accept`, `ui\_cancel`.



\---



\## 9. Component Pattern



Build reusable components as self-contained sub-scenes. A component owns its logic and exposes a clean API.



```gdscript

\# scripts/components/health\_component.gd

class\_name HealthComponent

extends Node



signal health\_changed(new\_health: float, max\_health: float)

signal died



@export var max\_health: float = 100.0



var current\_health: float:

&#x20;   get: return \_current\_health



var \_current\_health: float = 0.0



func \_ready() -> void:

&#x20;   \_current\_health = max\_health



func take\_damage(amount: float) -> void:

&#x20;   \_current\_health = maxf(0.0, \_current\_health - amount)

&#x20;   health\_changed.emit(\_current\_health, max\_health)

&#x20;   if \_current\_health == 0.0:

&#x20;       died.emit()



func heal(amount: float) -> void:

&#x20;   \_current\_health = minf(max\_health, \_current\_health + amount)

&#x20;   health\_changed.emit(\_current\_health, max\_health)

```



Common reusable components to build early:

\- `HealthComponent` — HP, damage, death signal

\- `HitboxComponent` / `HurtboxComponent` — combat collision

\- `StateMachineComponent` — generic FSM

\- `InteractableComponent` — player interaction trigger

\- `InventoryComponent` — item management

\- `TimerComponent` — managed cooldowns



\---



\## 10. Save \& Load System



```gdscript

\# autoloads/save\_manager.gd

class\_name SaveManager

extends Node



const SAVE\_PATH: String = "user://save.json"



func save(data: Dictionary) -> void:

&#x20;   var file := FileAccess.open(SAVE\_PATH, FileAccess.WRITE)

&#x20;   if file:

&#x20;       file.store\_string(JSON.stringify(data, "\\t"))



func load() -> Dictionary:

&#x20;   if not FileAccess.file\_exists(SAVE\_PATH):

&#x20;       return {}

&#x20;   var file := FileAccess.open(SAVE\_PATH, FileAccess.READ)

&#x20;   if file:

&#x20;       var result := JSON.parse\_string(file.get\_as\_text())

&#x20;       if result is Dictionary:

&#x20;           return result

&#x20;   return {}

```



For complex saves, use `ConfigFile` or serialize custom Resources with `ResourceSaver`.



\---



\## 11. Scene Transitions



```gdscript

\# autoloads/game\_manager.gd

func change\_scene(path: String, transition: bool = true) -> void:

&#x20;   if transition:

&#x20;       # Play transition animation before swapping

&#x20;       await \_transition\_player.play("fade\_out")

&#x20;   get\_tree().change\_scene\_to\_file(path)

&#x20;   if transition:

&#x20;       \_transition\_player.play("fade\_in")

```



Use a persistent `TransitionLayer` CanvasLayer in the autoload for fade/wipe effects.



\---



\## 12. Performance Guidelines



\- \*\*Object pooling\*\*: Pool frequently spawned/despawned objects (bullets, particles, enemies).

\- \*\*Signals over polling\*\*: Don't check state every frame — emit signals when state changes.

\- \*\*Avoid `get\_node()` in `\_process()`\*\*: Cache node references in `\_ready()` with `@onready`.

\- \*\*Use `call\_deferred()`\*\* when modifying the scene tree from within physics callbacks.

\- \*\*Visibility notifiers\*\*: Use `VisibleOnScreenNotifier2D/3D` to pause off-screen logic.

\- \*\*Groups for broadcast\*\*: Use `get\_tree().call\_group("enemies", "alert")` instead of looping manually.

\- \*\*Resource sharing\*\*: Share `.tres` resource instances between nodes when data is read-only.



\---



\## 13. Code Quality Checklist



Before finalizing any script:

\- \[ ] `class\_name` declared (if the class will be referenced elsewhere)

\- \[ ] All variables typed

\- \[ ] `@export` vars used for designer-tunable values

\- \[ ] `@onready` vars used instead of `get\_node()` calls

\- \[ ] No magic numbers — use named constants

\- \[ ] Signals used for cross-node communication (not direct parent calls)

\- \[ ] Signal callbacks named `\_on\_NodeName\_signal\_name`

\- \[ ] Private methods/vars prefixed with `\_`

\- \[ ] No business logic in `\_ready()` — use dedicated init methods

\- \[ ] No `print()` left in production code — use `push\_warning()` / `push\_error()`



\---



\## 14. Dimension-Specific Guidance



When the project is confirmed as 2D or 3D, read the appropriate reference:



\- \*\*2D projects\*\* → `references/godot-2d.md`  

&#x20; Covers: TileMap, Camera2D, physics layers, Sprite2D vs AnimatedSprite2D, parallax, pixel art settings.



\- \*\*3D projects\*\* → `references/godot-3d.md`  

&#x20; Covers: CSG vs MeshInstance3D, WorldEnvironment, lighting, CharacterBody3D, collision mesh best practices, cameras.



\---



\## 15. Recommended Addons



| Addon | Purpose |

|---|---|

| \[GUT](https://github.com/bitwes/Gut) | Unit testing framework |

| \[gdUnit4](https://github.com/MikeSchulze/gdUnit4) | Alternative test framework |

| \[Phantom Camera](https://github.com/ramokz/phantom-camera) | Advanced camera controller |

| \[Beehave](https://github.com/bitbra1n/beehave) | Behavior trees for AI |

| \[Dialogic](https://github.com/dialogic-godot/dialogic) | Dialogue system |



Always check the \[Godot Asset Library](https://godotengine.org/asset-library/asset) before writing custom tooling.



\---



\## Quick Reference: Common Patterns



```gdscript

\# Defer scene tree modifications

call\_deferred("queue\_free")



\# One-shot signal connection

node.signal\_name.connect(callback, CONNECT\_ONE\_SHOT)



\# Group-based broadcast

get\_tree().call\_group("enemies", "on\_player\_spotted", player\_position)



\# Typed scene instantiation

const BULLET\_SCENE: PackedScene = preload("res://scenes/entities/bullet/bullet.tscn")

func \_spawn\_bullet() -> void:

&#x20;   var bullet: Bullet = BULLET\_SCENE.instantiate()

&#x20;   get\_tree().current\_scene.add\_child(bullet)

&#x20;   bullet.global\_position = muzzle.global\_position



\# Await signal

await get\_tree().create\_timer(1.5).timeout



\# Property setter with validation

var health: float = 100.0:

&#x20;   set(value):

&#x20;       health = clampf(value, 0.0, max\_health)

&#x20;       health\_changed.emit(health)

```

