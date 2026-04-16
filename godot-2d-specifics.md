\# Godot 2D Reference



Supplement to the main `SKILL.md`. Read this when the project is confirmed as a 2D game.



\---



\## Node Hierarchy for 2D Games



```

World (Node2D)

├── TileMapLayer (terrain, platforms)

├── Entities (Node2D — container)

│   ├── Player (CharacterBody2D)

│   └── Enemies (Node2D — container)

├── Pickups (Node2D — container)

├── CanvasLayer (UI — always on top, unaffected by camera)

│   └── HUD.tscn

└── Camera2D

```



\---



\## Camera2D



```gdscript

\# Attach to player for follow cam, or to World and tween manually

@onready var \_camera: Camera2D = $Camera2D



func \_ready() -> void:

&#x20;   # Smooth follow

&#x20;   \_camera.position\_smoothing\_enabled = true

&#x20;   \_camera.position\_smoothing\_speed = 5.0

&#x20;   # Lock to level bounds

&#x20;   \_camera.limit\_left = 0

&#x20;   \_camera.limit\_right = 3200

&#x20;   \_camera.limit\_top = 0

&#x20;   \_camera.limit\_bottom = 1800

```



For complex camera behavior (room transitions, screenshake, cinematic locks), use the \*\*Phantom Camera\*\* addon.



\*\*Screenshake pattern:\*\*

```gdscript

\# In CameraController

func shake(duration: float, intensity: float) -> void:

&#x20;   var tween := create\_tween()

&#x20;   var elapsed := 0.0

&#x20;   while elapsed < duration:

&#x20;       \_camera.offset = Vector2(

&#x20;           randf\_range(-intensity, intensity),

&#x20;           randf\_range(-intensity, intensity)

&#x20;       )

&#x20;       await get\_tree().process\_frame

&#x20;       elapsed += get\_process\_delta\_time()

&#x20;   \_camera.offset = Vector2.ZERO

```



\---



\## TileMap / TileMapLayer (Godot 4.3+)



Godot 4.3 replaced `TileMap` with individual `TileMapLayer` nodes.



```gdscript

\# Reference a TileMapLayer

@onready var \_ground\_layer: TileMapLayer = $TileMapLayer



\# Get tile data at world position

func get\_tile\_at(world\_pos: Vector2) -> TileData:

&#x20;   var map\_pos := \_ground\_layer.local\_to\_map(world\_pos)

&#x20;   return \_ground\_layer.get\_cell\_tile\_data(map\_pos)



\# Check if a cell is solid (using custom data layer "solid")

func is\_solid(world\_pos: Vector2) -> bool:

&#x20;   var tile := get\_tile\_at(world\_pos)

&#x20;   return tile != null and tile.get\_custom\_data("solid")

```



\*\*TileSet best practices:\*\*

\- Define \*\*Custom Data Layers\*\* (e.g., `solid: bool`, `damage: float`, `surface\_type: String`) for gameplay logic.

\- Use \*\*Physics Layers\*\* in TileSet for collision, not custom data.

\- Use \*\*Terrain Sets\*\* for auto-tiling rather than manually placing edge tiles.



\---



\## CharacterBody2D Movement



```gdscript

extends CharacterBody2D



const GRAVITY: float = 980.0

const JUMP\_VELOCITY: float = -400.0



@export var stats: CharacterStats



func \_physics\_process(delta: float) -> void:

&#x20;   # Apply gravity

&#x20;   if not is\_on\_floor():

&#x20;       velocity.y += GRAVITY \* delta



&#x20;   # Horizontal movement

&#x20;   var direction := Input.get\_axis("move\_left", "move\_right")

&#x20;   velocity.x = direction \* stats.move\_speed



&#x20;   move\_and\_slide()



&#x20;   # Jump

&#x20;   if is\_on\_floor() and Input.is\_action\_just\_pressed("jump"):

&#x20;       velocity.y = JUMP\_VELOCITY

```



\---



\## Sprite \& Animation



\*\*Use `AnimatedSprite2D`\*\* for simple frame animation tied to the node itself.  

\*\*Use `Sprite2D` + `AnimationPlayer`\*\* when you need to animate multiple properties simultaneously (position, scale, color).



```gdscript

\# AnimatedSprite2D approach

@onready var \_sprite: AnimatedSprite2D = $AnimatedSprite2D



func \_update\_animation(direction: float, is\_grounded: bool) -> void:

&#x20;   if not is\_grounded:

&#x20;       \_sprite.play("jump")

&#x20;   elif absf(direction) > 0.1:

&#x20;       \_sprite.play("run")

&#x20;       \_sprite.flip\_h = direction < 0

&#x20;   else:

&#x20;       \_sprite.play("idle")

```



\*\*Pixel art settings\*\* (Project Settings):

\- `Rendering > Textures > Default Texture Filter` → \*\*Nearest\*\*

\- `Display > Window > Stretch > Mode` → \*\*canvas\_items\*\*

\- `Display > Window > Stretch > Aspect` → \*\*keep\*\*

\- Set base resolution to your pixel art resolution (e.g. 320×180), scale up at runtime.



\---



\## Physics Layers (2D)



Define layers in `Project Settings > Layer Names > 2D Physics`. Name them clearly:



| Layer | Name | Used by |

|---|---|---|

| 1 | world | TileMap, static terrain |

| 2 | player | Player body |

| 3 | enemy | Enemy bodies |

| 4 | player\_hitbox | Player attack areas |

| 5 | enemy\_hitbox | Enemy attack areas |

| 6 | pickup | Collectible items |

| 7 | projectile | Bullets, arrows |



Set `collision\_layer` (what this object IS) and `collision\_mask` (what this object DETECTS) carefully on every physics body and Area2D.



\---



\## Area2D for Hitbox / Hurtbox



```gdscript

\# scripts/components/hurtbox\_component.gd

class\_name HurtboxComponent

extends Area2D



signal hit(hitbox: HitboxComponent)



func \_ready() -> void:

&#x20;   area\_entered.connect(\_on\_area\_entered)



func \_on\_area\_entered(area: Area2D) -> void:

&#x20;   if area is HitboxComponent:

&#x20;       hit.emit(area)

```



```gdscript

\# scripts/components/hitbox\_component.gd

class\_name HitboxComponent

extends Area2D



@export var damage: float = 10.0

@export var knockback\_force: float = 200.0

```



Wire them in the entity's script:

```gdscript

@onready var \_hurtbox: HurtboxComponent = $HurtboxComponent

func \_ready() -> void:

&#x20;   \_hurtbox.hit.connect(\_on\_hit)

func \_on\_hit(hitbox: HitboxComponent) -> void:

&#x20;   health\_component.take\_damage(hitbox.damage)

```



\---



\## Parallax Background



```

ParallaxBackground

├── ParallaxLayer (motion\_scale = (0.1, 0.0))  ← Far sky

│   └── Sprite2D (clouds far)

├── ParallaxLayer (motion\_scale = (0.3, 0.1))  ← Mid mountains

│   └── Sprite2D (mountains)

└── ParallaxLayer (motion\_scale = (0.6, 0.2))  ← Near trees

&#x20;   └── Sprite2D (trees)

```



Set `ParallaxBackground` as a child of the main camera's canvas, or as a direct scene child — it follows the camera automatically.



\---



\## UI in 2D Games



Always place HUD/menus in a `CanvasLayer` (layer `1` or higher) so they are unaffected by the game camera.



```

CanvasLayer (layer = 1)

└── HUD (Control)

&#x20;   ├── HealthBar (TextureProgressBar)

&#x20;   ├── ScoreLabel (Label)

&#x20;   └── MiniMap (SubViewportContainer)

```



For menus (pause, main menu, game over), use a separate `CanvasLayer` or a dedicated scene loaded additively.



\---



\## Y-Sort for Top-Down Games



For top-down 2D, enable \*\*Y-Sort\*\* on container nodes so sprites behind appear behind:



```gdscript

\# On the entities container node:

\# Inspector: Y Sort Enabled = true



\# Each entity's origin must be at their feet for correct sorting.

```

