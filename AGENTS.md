# AGENTS.md — OpenDungeonsPlus

## Project Overview

OpenDungeonsPlus is an open source real-time strategy game inspired by Dungeon Keeper and Evil Genius. Players build underground dungeons inhabited by creatures, fight for territory, cast spells, and lay traps. It is a community-driven fork/continuation of the original OpenDungeons project.

- **Language**: C++14
- **License**: GPLv3+
- **Version**: 0.7.1
- **Binary**: `opendungeons-plus`
- **Repo**: https://github.com/tomluchowski/OpenDungeonsPlus

## Build System

### Two-Level CMake

The project uses a parent-child CMake structure:

```
CMakeLists.txt          (outer — builds dependencies as ExternalProjects)
OpenDungeonsPlus/
  CMakeLists.txt        (inner — builds the game itself)
```

The outer CMakeLists.txt fetches and builds dependencies from source via `ExternalProject_Add`:
- **pybind11** — Python/C++ bindings
- **OIS** — input handling
- **OGRE** — 3D rendering engine (OpenGL3+, GLES2 renderers)
- **CEGUI** — GUI system (Crazy Eddie's GUI, v0-tomluchowski-14.x fork)

The inner CMakeLists.txt finds the pre-built dependencies and compiles the game.

### Build Commands

```bash
# From the repo root (builds everything including deps):
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug -DOD_TREAT_WARNINGS_AS_ERRORS=OFF
make -j$(nproc)

# To build only the game (deps already installed):
cd OpenDungeonsPlus
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug -DOD_TREAT_WARNINGS_AS_ERRORS=OFF
make -j$(nproc)
```

### Key CMake Options

| Option | Default | Description |
|--------|---------|-------------|
| `OD_TREAT_WARNINGS_AS_ERRORS` | ON | Treat compiler warnings as errors |
| `OD_ENABLE_WARNINGS` | ON | Enable verbose warning flags |
| `OD_USE_SFML_WINDOW` | OFF | Use SFML for window handling (instead of OGRE) |
| `OD_BUILD_TESTING` | OFF | Build unit tests (requires Boost.Test) |
| `OD_DATA_PATH` | varies | Path to game data files |
| `OD_PLUGINS_CFG_PATH` | varies | Path to Ogre plugins.cfg |

### Dependencies

Required libraries found via `find_package`:
- **OGRE** >= 1.9.0 (with RTShaderSystem, Overlay, Bites components)
- **CEGUI** >= 0.8.0 (with OgreRenderer)
- **SFML** >= 2.0 (Audio, System, Network; optionally Window/Graphics)
- **OIS** — Object-oriented Input System
- **Boost** — filesystem, locale, program_options, thread
- **Python** + **pybind11** — embedded Python scripting
- **Threads** (pthreads)

### Run-In-Place

When building outside the source directory, CMake creates symlinks for asset directories (`config`, `gui`, `levels`, `materials`, `models`, `music`, `particles`, `python`, `scripts`, `shaders`, `sounds`) so the binary can run from the build directory.

## Source Code Structure

All source is under `OpenDungeonsPlus/source/`. The entry point is `main.cpp` and `ODApplication.cpp`.

```
source/
  main.cpp                   — Entry point, parses CLI options
  ODApplication.cpp/.h       — Game startup, client/server initialization

  ai/                        — AI system
    AIFactory                 — Factory for AI types
    AIManager                 — Manages AI instances for seats
    BaseAI                    — Abstract AI interface
    KeeperAI                  — Keeper AI implementation
    KeeperAIType              — AI difficulty type enum

  camera/                    — Camera and culling
    CameraManager             — Camera positioning and movement
    HermiteCatmullSpline      — Spline interpolation for camera paths
    CullingManager            — Tile/entity visibility culling
    SlopeWalk                 — Terrain slope traversal

  creatureaction/            — Creature actions (individual tasks)
    CreatureAction            — Base class for all actions
    CreatureActionFight       — Engage in combat
    CreatureActionDigTile     — Dig earth/claimed tiles
    CreatureActionClaimTile   — Claim ground/wall tiles
    CreatureActionSleep       — Rest in dormitory
    CreatureActionEatChicken  — Eat from hatchery
    ... (20+ action types)

  creaturebehaviour/         — Creature behaviours (decision-making)
    CreatureBehaviour         — Base class
    CreatureBehaviourAttackEnemy — Attack nearby enemies
    CreatureBehaviourFleeWhenWeak — Flee when low HP
    CreatureBehaviourManager  — Manages behaviour assignments

  creatureeffect/            — Creature effects (buffs/debuffs)
    CreatureEffect            — Base class
    CreatureEffectHeal        — HP restoration
    CreatureEffectSpeedChange — Speed modification
    CreatureEffectStrengthChange — Attack strength modification
    CreatureEffectExplosion   — Explosion damage
    CreatureEffectManager     — Manages active effects

  creaturemood/              — Creature mood system
    CreatureMood              — Base class
    CreatureMoodHunger        — Affected by hunger level
    CreatureMoodWakefulness   — Affected by sleep
    CreatureMoodFee           — Affected by pay satisfaction
    CreatureMoodManager       — Manages mood modifiers

  creatureskill/             — Creature skills (combat abilities)
    CreatureSkill             — Base class
    CreatureSkillMeleeFight   — Melee attack
    CreatureSkillMissileLaunch — Ranged attack
    CreatureSkillHealSelf     — Self-healing
    CreatureSkillManager      — Manages skill execution

  entities/                  — Game entities
    GameEntity                — Base entity class
    MovableGameEntity         — Base for movable entities
    RenderedMovableEntity     — Base for 3D rendered entities
    Creature                  — Creature entity (workers, fighters)
    CreatureDefinition        — Creature configuration/balancing
    Building                  — Room building tiles
    BuildingObject            — Room furniture/objects
    Tile                      — Game map tile
    MapLight                  — Map light source
    DoorEntity                — Doors
    TrapEntity                — Trap base
    CraftedTrap               — Crafted trap instance
    ChickenEntity             — Hatchery chicken
    TreasuryObject            — Gold pile in treasury
    MissileBoulder/OneHit/Object — Projectile types
    Weapon                    — Creature equipment
    PersistentObject          — Objects that persist in saves
    EntityLoading             — Entity deserialization from config

  eventsystem/               — Event system (observer pattern)
    Event                     — Base event
    Subject/Observer          — Observer pattern implementation
    EventHandler              — Event dispatch

  game/                      — Game state
    Player                    — Human player controller
    PlayerSelection           — Selection state
    Seat                      — Faction/team state
    Skill/SkillManager        — Research/skill system

  gamemap/                   — Map and tiles
    GameMap                   — The entire game map
    MapHandler                — Map loading and saving
    TileContainer             — Tile collection
    TileSet                   — Biome/tileset definitions
    MiniMap/MiniMapDrawn/MiniMapCamera — Minimap variants
    DraggableTileContainer    — Map panning
    Pathfinding               — A* pathfinding

  goals/                     — Win/loss conditions
    Goal                      — Base class
    GoalKillAllEnemies        — Eliminate all opponents
    GoalMineNGold             — Accumulate gold
    GoalClaimNTiles           — Claim territory
    GoalProtectCreature       — Keep a creature alive
    GoalProtectDungeonTemple  — Protect the dungeon temple

  modes/                     — Game modes and menus
    AbstractApplicationMode   — Base mode class
    ModeManager               — Mode stack manager
    GameMode                  — In-game mode
    EditorMode                — Level editor
    MenuModeMain              — Main menu
    MenuModeSkirmish          — Skirmish setup
    MenuModeMultiplayerClient/Server — Multiplayer setup
    InputManager/InputBridge  — Input handling
    ConsoleInterface          — In-game console
    SettingsWindow            — Settings UI

  network/                   — Multiplayer networking
    ODServer/ODClient         — Game server/client logic
    ODSocketServer/ODSocketClient — Low-level socket I/O
    ODPacket                  — Network packet serialization
    ServerNotification/ClientNotification — Sync messages

  render/                    — Rendering
    RenderManager             — OGRE scene management
    ODFrameListener           — Per-frame rendering callback
    Gui                       — GUI management (CEGUI)
    TextRenderer              — Text overlay rendering
    CreatureOverlayStatus     — Creature status overlays
    DebugDrawer               — Debug visualizations

  renderscene/               — Scripted render scenes (main menu, cutscenes)
    RenderScene               — Base scene command
    RenderSceneAddEntity      — Spawn entity in scene
    RenderSceneCameraMove     — Animate camera
    RenderSceneAnimationOnce/Time — Play animations
    RenderSceneManager        — Scene command runner

  rooms/                     — Room types
    Room                      — Base class
    RoomTreasury              — Gold storage, payday
    RoomDormitory             — Creature sleep/rest
    RoomTrainingHall          — Creature XP training
    RoomLibrary               — Research/skills
    RoomWorkshop              — Trap crafting
    RoomHatchery              — Food (chickens)
    RoomPortal                — Creature spawning
    RoomDungeonTemple         — Core dungeon structure
    RoomArena                 — Creature combat arena
    RoomCasino                — Creature recreation
    RoomPrison                — Convert enemies to skeletons
    RoomTorture               — Torture enemies
    RoomCrypt                 — Decaying corpses
    RoomBridge/BridgeStone/BridgeWooden — Bridges
    RoomManager               — Room lifecycle manager

  sound/                     — Audio
    MusicPlayer               — Background music
    SoundEffectsManager       — Sound effect triggers

  spawnconditions/           — Creature spawn prerequisites
    SpawnCondition            — Base class
    SpawnConditionCreature/Gold/Room — Specific conditions

  spells/                    — Keeper spells
    Spell                     — Base class
    SpellSummonWorker         — Summon imp/worker
    SpellCallToWar            — Rally fighters
    SpellCreatureHeal/Haste/Defense/Strength/Slow/Weak/Explosion — Buffs
    SpellEyeEvil              — Vision spell
    SpellManager              — Spell lifecycle

  traps/                     — Trap types
    Trap                      — Base class
    TrapBoulder/Cannon/Spike/Door — Specific traps
    TrapManager               — Trap lifecycle

  utils/                     — Utilities
    ConfigManager             — INI-style config parsing
    ResourceManager           — Asset loading and management
    LogManager/LogSink*       — Logging infrastructure
    Random                    — RNG utilities
    FrameRateLimiter          — Frame pacing
    Helper                    — Misc helpers
    MasterServer              — Master server communication
    VectorInt64               — 2D integer vector
    StackTrace*               — Platform-specific backtraces
```

## Data Files

```
config/          — Game configuration (creatures, rooms, traps, spells, equipment, skills, etc.)
gui/             — CEGUI layouts, imagesets, fonts, schemes
levels/          — Level/map files (singleplayer, multiplayer, skirmish)
materials/       — OGRE material scripts and textures
models/          — 3D mesh files (.mesh)
music/           — Background music (OGG)
particles/       — Particle effect definitions
python/          — Embedded Python scripts
scripts/         — Packaging and CI helper scripts
shaders/         — GPU shader programs
sounds/          — Sound effects (OGG/WAV)
```

## Configuration System

Game balance and behaviour is driven by INI-style config files in `config/`. Key files:

- `creatures.cfg` — Creature definitions (stats, skills, equipment, spawn pool)
- `rooms.cfg` — Room type definitions (cost, capacity, spawn conditions)
- `traps.cfg` — Trap type definitions (damage, cost, crafting requirements)
- `spells.cfg` — Spell definitions (cooldown, cost, effects)
- `equipment.cfg` — Weapon/armor definitions
- `skills.cfg` — Skill/research tree
- `seats.cfg` — Faction configurations
- `global.cfg` — Global game settings

## Key Architecture Patterns

### Entity Hierarchy
```
GameEntity
  └── MovableGameEntity
        └── RenderedMovableEntity
              ├── Creature
              ├── Building
              ├── DoorEntity
              ├── TrapEntity → CraftedTrap
              ├── ChickenEntity
              ├── TreasuryObject
              ├── MissileObject → MissileBoulder, MissileOneHit
              └── ...
  └── Tile
  └── MapLight
```

### Action-Behaviour-Mood
Creatures are driven by a three-layer AI system:
1. **Moods** — Passive modifiers (hunger, sleepiness, pay satisfaction) generate needs
2. **Behaviours** — Decision-making (attack enemies, flee when weak)
3. **Actions** — Concrete tasks (walk to tile, dig, fight, eat, sleep)

### Mode System
Game screens are managed as a stack of `AbstractApplicationMode` instances:
- `MenuModeMain` (top level)
- `MenuModeSkirmish`, `MenuModeMultiplayerClient`, etc. (sub-menus)
- `GameMode` (in-game)
- `EditorMode` (level editor)
- `ConsoleInterface` (can push onto any mode)

### Network Model
Client-server architecture:
- Server runs the authoritative game simulation
- Clients send input commands, receive state updates
- `ODServer`/`ODClient` implement the game protocol
- `ODSocketServer`/`ODSocketClient` handle TCP socket I/O
- `ODPacket` handles serialization

## Coding Conventions

- **C++ Standard**: C++14
- **Indentation**: 4 spaces (tabs in some files — match surrounding style)
- **Naming**: CamelCase classes (`CreatureAction`), camelBack methods (`doAction()`), UPPER_CASE constants
- **Headers**: `.h` extension, include guards use `#ifndef CLASSNAME_H` / `#define CLASSNAME_H`
- **License header**: Every source file has a GPLv3+ header block
- **Copyright**: "Copyright (C) 2011-2016 OpenDungeons Team"
- **Documentation**: Doxygen-style `\brief`, `\file`, `\author`, `\date` tags
- **Warnings**: Treat warnings as errors by default (`-Wall` + `-Werror` in CMake)
- **Memory**: Raw `new`/`delete` with manual ownership; no smart pointers in most code

## Key Files for Common Tasks

| Task | Key Files |
|------|-----------|
| Add a creature type | `config/creatures.cfg`, `entities/CreatureDefinition.cpp` |
| Add a room type | `config/rooms.cfg`, `rooms/Room*.cpp`, `rooms/RoomManager.cpp` |
| Add a spell | `config/spells.cfg`, `spells/Spell*.cpp`, `spells/SpellManager.cpp` |
| Add a trap type | `config/traps.cfg`, `traps/Trap*.cpp`, `traps/TrapManager.cpp` |
| Add a creature action | `creatureaction/CreatureAction*.cpp`, register in `Creature.cpp` |
| Add a behaviour | `creaturebehaviour/CreatureBehaviour*.cpp`, `CreatureBehaviourManager.cpp` |
| Add a level | `levels/*.level`, add to `levels/` directory |
| Modify UI | `gui/*.layout`, `render/Gui.cpp` |
| Modify config parsing | `utils/ConfigManager.cpp` |
| Add a new source file | Add to `OD_SOURCEFILES` list in `CMakeLists.txt` |

## Editor

The game includes a level editor (`EditorMode`) that allows creating and modifying maps. It is built on top of the game mode (`GameEditorModeBase`) and includes a console interface (`GameEditorModeConsole`).

## Testing

- Unit tests use Boost.Test framework
- Enable with `-DOD_BUILD_TESTING=ON -DBUILD_TESTING=ON`
- Test sources in `source/tests/`
