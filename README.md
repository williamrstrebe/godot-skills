# godot-skills

Experimental notes and guidelines for working with Godot.

## Structure

* `main-godot-skill.md` — Dimension-agnostic systems and core architecture
* `godot-2d.md` — 2D-specific patterns and components
* `godot-3d.md` — 3D-specific patterns and components

## Purpose

This repository serves as a structured reference for organizing knowledge about Godot, focusing on scalable architecture and reusable patterns across both 2D and 3D projects.

### Core topics covered (main)

* Canonical project folder structure
* Node and scene composition patterns
* Full GDScript standards (typed usage, section ordering, naming conventions)
* Signals (emit upward / call downward principle)
* Autoloads and EventBus pattern
* Custom Resources for data-driven design
* State Machine component pattern
* Input Map usage
* Reusable component pattern (e.g., HealthComponent, Hitbox)
* Save/load system
* Scene transitions
* Performance guidelines
* Code quality checklist

### 2D-specific topics

* Camera2D (including screenshake)
* TileMapLayer usage
* CharacterBody2D patterns
* AnimatedSprite2D workflows
* Parallax backgrounds
* Physics layers and masks
* Area2D (hitbox/hurtbox patterns)
* Y-Sort usage
* Pixel art configuration

### 3D-specific topics

* CharacterBody3D movement patterns
* First-person and third-person camera setups
* Mesh and collision best practices
* Lighting and WorldEnvironment setup
* NavigationAgent3D pathfinding
* AnimationTree blending
* LOD and MultiMesh performance strategies
* World-space UI

Content is exploratory and may evolve over time.

## License

Apache 2.0
