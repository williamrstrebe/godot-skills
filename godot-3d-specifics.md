\# Godot 3D Reference



Supplement to the main `SKILL.md`. Read this when the project is confirmed as a 3D game.



\---



\## Node Hierarchy for 3D Games



```

World (Node3D)

├── WorldEnvironment          ← Sky, fog, ambient light

├── DirectionalLight3D        ← Sun

├── StaticBody3D (terrain)

│   ├── MeshInstance3D

│   └── CollisionShape3D

├── Entities (Node3D — container)

│   ├── Player (CharacterBody3D)

│   └── Enemies (Node3D — container)

└── NavigationRegion3D        ← AI pathfinding bake zone

```



\---



\## CharacterBody3D Movement



```gdscript

extends CharacterBody3D



const GRAVITY: float = -9.8



@export var stats: CharacterStats

@export var camera\_pivot: Node3D



func \_physics\_process(delta: float) -> void:

&#x20;   # Gravity

&#x20;   if not is\_on\_floor():

&#x20;       velocity.y += GRAVITY \* delta



&#x20;   # Input relative to camera direction

&#x20;   var input\_dir := Input.get\_vector("move\_left", "move\_right", "move\_up", "move\_down")

&#x20;   var cam\_basis := camera\_pivot.global\_transform.basis

&#x20;   var direction := (cam\_basis \* Vector3(input\_dir.x, 0, input\_dir.y)).normalized()

&#x20;   direction.y = 0.0



&#x20;   velocity.x = direction.x \* stats.move\_speed

&#x20;   velocity.z = direction.z \* stats.move\_speed



&#x20;   move\_and\_slide()



&#x20;   # Face movement direction

&#x20;   if direction.length() > 0.1:

&#x20;       var target\_basis := Basis.looking\_at(direction)

&#x20;       global\_transform.basis = global\_transform.basis.slerp(target\_basis, 10.0 \* delta)



&#x20;   # Jump

&#x20;   if is\_on\_floor() and Input.is\_action\_just\_pressed("jump"):

&#x20;       velocity.y = stats.jump\_force

```



\---



\## Camera in 3D



\### First-Person Camera

```

Player (CharacterBody3D)

└── Head (Node3D)          ← Rotates on X (pitch)

&#x20;   └── Camera3D

```



```gdscript

\# first\_person\_camera.gd

@export var sensitivity: float = 0.002

@onready var \_head: Node3D = $Head



func \_input(event: InputEvent) -> void:

&#x20;   if event is InputEventMouseMotion and Input.mouse\_mode == Input.MOUSE\_MODE\_CAPTURED:

&#x20;       rotate\_y(-event.relative.x \* sensitivity)

&#x20;       \_head.rotate\_x(-event.relative.y \* sensitivity)

&#x20;       \_head.rotation.x = clampf(\_head.rotation.x, deg\_to\_rad(-89), deg\_to\_rad(89))

```



\### Third-Person Camera (Spring Arm)

```

Player (CharacterBody3D)

└── CameraPivot (Node3D)       ← Rotates with mouse

&#x20;   └── SpringArm3D            ← Prevents clipping

&#x20;       └── Camera3D

```



```gdscript

@onready var \_pivot: Node3D = $CameraPivot

@onready var \_spring\_arm: SpringArm3D = $CameraPivot/SpringArm3D



func \_input(event: InputEvent) -> void:

&#x20;   if event is InputEventMouseMotion:

&#x20;       \_pivot.rotate\_y(-event.relative.x \* sensitivity)

&#x20;       \_spring\_arm.rotate\_x(-event.relative.y \* sensitivity)

&#x20;       \_spring\_arm.rotation.x = clampf(\_spring\_arm.rotation.x, deg\_to\_rad(-60), deg\_to\_rad(20))

```



\---



\## Mesh \& Collision Best Practices



| Use case | Node |

|---|---|

| Visible complex mesh | `MeshInstance3D` |

| Static level geometry | `StaticBody3D` + `CollisionShape3D` |

| Moving platforms | `AnimatableBody3D` |

| Player / enemies | `CharacterBody3D` |

| Trigger volumes | `Area3D` + `CollisionShape3D` |

| Prototype / blockout | `CSGBox3D`, `CSGCylinder3D` (dev only) |



\*\*Never use `ConcavePolygonShape3D` for moving bodies\*\* — it's only valid on `StaticBody3D`. Use `ConvexPolygonShape3D` or a compound of primitive shapes for dynamic objects.



\*\*Collision mesh\*\* tips:

\- Keep collision geometry simple (fewer polygons = faster simulation).

\- Use `CollisionShape3D` with primitive shapes (`BoxShape3D`, `CapsuleShape3D`, `SphereShape3D`) over mesh shapes whenever possible.

\- Generate `ConvexPolygonShape3D` from mesh in the editor for mid-complexity shapes.



\---



\## Physics Layers (3D)



Define in `Project Settings > Layer Names > 3D Physics`:



| Layer | Name |

|---|---|

| 1 | world |

| 2 | player |

| 3 | enemy |

| 4 | player\_hitbox |

| 5 | enemy\_hitbox |

| 6 | pickup |

| 7 | projectile |

| 8 | ragdoll |



\---



\## Lighting \& WorldEnvironment



Always set up a `WorldEnvironment` node with a sky for realistic ambient light.



```

WorldEnvironment

&#x20;   environment: Environment resource

&#x20;       └── Background: Sky (ProceduralSkyMaterial or HDR panorama)

&#x20;       └── Ambient Light: Sky mode (picks from sky color)

&#x20;       └── Tonemap: Filmic or ACES

&#x20;       └── SSAO, SSIL: Enable for quality

&#x20;       └── Glow: Optional

&#x20;       └── Fog: Depth fog for atmosphere

```



\*\*Light types:\*\*

\- `DirectionalLight3D` — Sun / moon. Use `shadow\_mode = PCSS` for soft shadows.

\- `OmniLight3D` — Point light (lamps, fires). Bake to lightmap for static.

\- `SpotLight3D` — Flashlights, spotlights.



\*\*For performance:\*\*

\- Bake static lighting with `LightmapGI` for indoor/static scenes.

\- Use `VoxelGI` for dynamic GI in medium scenes.

\- Use `ReflectionProbe` for local reflections (interiors, puddles).

\- Limit dynamic shadow-casting lights to 1–3 max per frame.



\---



\## Navigation \& Pathfinding (3D)



```gdscript

\# AI enemy script

@onready var \_nav\_agent: NavigationAgent3D = $NavigationAgent3D



func \_ready() -> void:

&#x20;   \_nav\_agent.velocity\_computed.connect(\_on\_velocity\_computed)



func set\_target(pos: Vector3) -> void:

&#x20;   \_nav\_agent.target\_position = pos



func \_physics\_process(delta: float) -> void:

&#x20;   if \_nav\_agent.is\_navigation\_finished():

&#x20;       return

&#x20;   var next\_pos := \_nav\_agent.get\_next\_path\_position()

&#x20;   var direction := (next\_pos - global\_position).normalized()

&#x20;   var desired\_velocity := direction \* stats.move\_speed

&#x20;   \_nav\_agent.set\_velocity(desired\_velocity)



func \_on\_velocity\_computed(safe\_velocity: Vector3) -> void:

&#x20;   velocity = safe\_velocity

&#x20;   move\_and\_slide()

```



\*\*Bake navigation mesh:\*\*

\- Add `NavigationRegion3D` to level root.

\- Set `NavigationMesh` resource on it (configure agent radius, height, max slope).

\- Click "Bake NavigationMesh" in editor — or use `NavigationServer3D.bake\_from\_source\_geometry\_data()` at runtime for procedural levels.



\---



\## Animation in 3D



Use \*\*`AnimationPlayer`\*\* for scripted animations and cutscenes.  

Use \*\*`AnimationTree`\*\* with `AnimationNodeStateMachine` for blended character locomotion.



```

Player (CharacterBody3D)

├── Skeleton3D

│   └── (bone hierarchy)

├── MeshInstance3D (skin assigned to skeleton)

├── AnimationPlayer        ← Raw animation clips

└── AnimationTree          ← State machine blending

```



```gdscript

@onready var \_anim\_tree: AnimationTree = $AnimationTree



func \_ready() -> void:

&#x20;   \_anim\_tree.active = true



func \_update\_animation(speed: float, is\_grounded: bool) -> void:

&#x20;   \_anim\_tree.set("parameters/blend\_position", speed / stats.move\_speed)

&#x20;   \_anim\_tree.set("parameters/conditions/is\_grounded", is\_grounded)

```



\---



\## LOD \& Performance (3D)



\- Use `VisibleOnScreenNotifier3D` to disable enemy AI when off-screen.

\- Enable \*\*LOD\*\* on `MeshInstance3D` for detailed meshes (set LOD bias in project settings).

\- Use \*\*MultiMeshInstance3D\*\* for large numbers of identical objects (grass, trees, rocks).

\- Use \*\*Object Pooling\*\* for projectiles and particle-spawned objects (same as 2D).

\- Profile with \*\*Godot's built-in profiler\*\* and the \*\*RenderingServer monitor\*\*.



```gdscript

\# MultiMesh for instanced foliage/props

@onready var \_multi\_mesh: MultiMeshInstance3D = $MultiMeshInstance3D



func \_spawn\_grass(positions: Array\[Vector3]) -> void:

&#x20;   \_multi\_mesh.multimesh.instance\_count = positions.size()

&#x20;   for i in positions.size():

&#x20;       var t := Transform3D(Basis(), positions\[i])

&#x20;       \_multi\_mesh.multimesh.set\_instance\_transform(i, t)

```



\---



\## Shaders in 3D



Prefer `StandardMaterial3D` for most surfaces. Use `ShaderMaterial` only when custom visual effects are needed.



```gdscript

\# Swap material at runtime

@onready var \_mesh: MeshInstance3D = $MeshInstance3D

@export var hit\_material: ShaderMaterial



func flash\_hit() -> void:

&#x20;   var original := \_mesh.get\_surface\_override\_material(0)

&#x20;   \_mesh.set\_surface\_override\_material(0, hit\_material)

&#x20;   await get\_tree().create\_timer(0.1).timeout

&#x20;   \_mesh.set\_surface\_override\_material(0, original)

```



\---



\## 3D UI (World-Space)



For in-world labels, health bars above enemies, or floating UI:



```

Enemy (CharacterBody3D)

└── SubViewport (size: 256x64)

&#x20;   └── HealthBarUI (Control)

```



Or simpler: use `Label3D` for floating text, `Sprite3D` with `billboard = Enabled` for 2D sprites in 3D space.

