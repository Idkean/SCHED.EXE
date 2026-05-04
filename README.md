# SCHED.EXE — GDScript Source Code
## Demo Build | City College of Calamba | 2025

---

## Project Structure

```
schedexe/
├── autoloads/
│   ├── game_state_manager.gd    ← Central game brain (register as "GameStateManager")
│   └── gantt_chart_data.gd      ← Gantt chart event store (register as "GanttChartData")
│
├── scripts/
│   ├── algorithms/
│   │   ├── i_scheduler.gd       ← Base scheduler interface
│   │   ├── fcfs_scheduler.gd    ← First Come First Served
│   │   ├── sjf_scheduler.gd     ← Shortest Job First
│   │   └── priority_scheduler.gd← Priority Scheduling
│   │
│   ├── core/
│   │   ├── process_data.gd      ← Data class for one process/enemy
│   │   ├── process_queue.gd     ← Queue manager with sorted access
│   │   ├── wave_data.gd         ← Resource: defines one wave
│   │   ├── spawn_entry.gd       ← Resource: one enemy spawn instruction
│   │   ├── wave_manager.gd      ← Spawns enemies, tracks wave progress
│   │   ├── results_calculator.gd← Computes WT, TAT, CPU utilization
│   │   └── wave_factory.gd      ← Builds all 8 WaveData objects in code
│   │
│   ├── entities/
│   │   ├── enemy.gd             ← Enemy/process node (CharacterBody2D)
│   │   └── student_defender.gd  ← Tower node (Node2D)
│   │
│   └── ui/
│       └── gantt_chart.gd       ← Live scrolling Gantt chart (Control)
│
└── scenes/
    ├── main_menu.gd
    ├── briefing.gd
    ├── deploy.gd
    ├── execution.gd
    ├── results.gd
    ├── game_over.gd
    └── completion.gd
```

---

## Setup Steps in Godot 4

### 1. Register Autoloads
Go to **Project > Project Settings > Autoload** and add:

| Path | Name |
|------|------|
| `res://autoloads/game_state_manager.gd` | `GameStateManager` |
| `res://autoloads/gantt_chart_data.gd`   | `GanttChartData`   |

---

### 2. Create Your Scenes
Create a `.tscn` file for each scene listed below and attach the matching script.

| Scene file               | Root node type   | Script            |
|--------------------------|------------------|-------------------|
| `main_menu.tscn`         | Control          | main_menu.gd      |
| `briefing.tscn`          | Control          | briefing.gd       |
| `deploy.tscn`            | Control          | deploy.gd         |
| `execution.tscn`         | Node2D           | execution.gd      |
| `results.tscn`           | Control          | results.gd        |
| `game_over.tscn`         | Control          | game_over.gd      |
| `completion.tscn`        | Control          | completion.gd     |

Set `main_menu.tscn` as the **Main Scene** in Project Settings.

---

### 3. Create the Enemy Scene
1. Create a new scene: **CharacterBody2D** as root
2. Add children:
   - `Sprite2D` — your enemy sprite
   - `CollisionShape2D` — capsule or rect shape
   - `ProgressBar` — name it `HealthBar`, anchor above sprite
   - `Label` — name it `Label`, shows process name
3. Attach `enemy.gd` to the root
4. Save as `res://scenes/enemies/enemy.tscn`

---

### 4. Create the Student Defender Scene
1. Create a new scene: **Node2D** as root
2. Add children:
   - `Sprite2D` — student sprite
   - `AnimationPlayer` — add an "attack" animation
   - `Area2D` — name it `RangeArea` (collision shape added in code)
   - `Label` — name it `Label`, shows algorithm name
3. Attach `student_defender.gd` to the root
4. Save as `res://scenes/defenders/student_defender.tscn`

---

### 5. Create the Execution Scene
The execution scene needs these specific node names:

```
Node2D  [execution.gd]
├── WaveManager            [wave_manager.gd]
├── LaneNodes
│   ├── LaneStart0         Marker2D — where enemies spawn
│   ├── LaneStart1         Marker2D
│   └── LaneStart2         Marker2D
├── DefenderNodes
│   ├── LaneNode0          Marker2D — where students stand
│   ├── LaneNode1          Marker2D
│   └── LaneNode2          Marker2D
└── CanvasLayer
    └── HUD
        ├── LabHPBar        ProgressBar (top of screen)
        ├── WaveLabel       Label
        └── GanttChart      Control [gantt_chart.gd] — bottom 120px strip
```

In the WaveManager node Inspector, assign the 3 LaneStart Marker2D paths
to the `lane_starts` array.

---

### 6. Load Waves via WaveFactory
In `game_state_manager.gd`, replace `_load_wave_resources()` with:

```gdscript
func _load_wave_resources() -> void:
    var enemy_scene = load("res://scenes/enemies/enemy.tscn")
    wave_resources = WaveFactory.build_all_waves(enemy_scene)
```

Or create `.tres` files manually in the Inspector using `WaveData` and `SpawnEntry`.

---

### 7. Assign student_scene in Execution Scene
In the Execution scene's Inspector (execution.gd exported var),
drag `student_defender.tscn` into the `student_scene` slot.

---

## Build Order (recommended)

1. Set up autoloads
2. Create Enemy scene + test it moving across screen
3. Create StudentDefender scene + test it dealing damage
4. Wire up execution.gd — get one wave spawning and defeating enemies
5. Verify Gantt chart draws correctly
6. Build Briefing and Deploy UI scenes
7. Build Results scene and verify WT/TAT math
8. Wire GameStateManager phase transitions end-to-end
9. Add all 8 waves via WaveFactory
10. Art + audio pass

---

## Algorithm Color Reference

| Algorithm | Color   | Hex       |
|-----------|---------|-----------|
| FCFS      | Blue    | `#4A90D9` |
| SJF       | Green   | `#27AE60` |
| PRIORITY  | Red     | `#E74C3C` |

---

## Tips

- Use `print(ResultsCalculator.calculate(...))` in `_on_wave_complete` to verify
  your math before building the Results UI.
- The Gantt chart uses `queue_redraw()` every frame — this is intentional.
  It is lightweight for a 2D custom draw call.
- `WaveFactory` uses `delay` (seconds after wave start) to stagger spawns,
  not `arrival_time`. Arrival time is the process data used for scheduling math.
  Make sure `arrival_time ≈ delay` for consistent results.
- Student defenders only activate after `is_active = true` is set in execution.gd.
  This gives you a window to add a countdown timer before combat starts.
