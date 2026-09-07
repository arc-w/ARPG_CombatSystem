# ARPG Combat System

A modular Action RPG and Soulslike combat framework built in **Unreal Engine 5**. The system is designed around strict state validation, decoupled collision tracing, and a centralized enum-driven state machine to provide deterministic combat behavior and prevent conflicting inputs.

---

## Combat Mechanics

### Block

* Blocks incoming attacks until the player's **Posture** is broken.
* Once Posture breaks, the player becomes **Stunned**.

### Parry

* A perfectly timed Block provides a **0.3-second window of invulnerability**.
* Successfully timing the Block against an incoming attack results in a **Parry**.

### Light Attack

* Deals **normal damage** to the target.
* Deals **Posture damage** when hitting an opponent who is blocking.

### Heavy Attack

* Deals **high damage**.
* When blocked, it **immediately breaks the opponent's Posture**.

---

## Technical Architecture

### State Machine (`E_PlayerState`)

The combat framework uses a centralized state machine based on the `E_PlayerState` enumeration:

`Idle`, `Attacking`, `Blocking`, `Parrying`, `Dodging`, `Stunned`, `Dead`

* **Input Validation:** Movement and combat inputs are validated against the current state before being routed to movement components or animation montage execution.
* **Concurrency Control:** Prevents conflicting actions and state corruption, such as attempting to attack while stunned or processing multiple combat state transitions simultaneously.

### Input Handling

* Built entirely with the **Enhanced Input System**.
* Input Mapping Contexts (`IMC_Default`, `IMC_MouseLook`) are dynamically configured at runtime through the `PlayerController`.
* Hardware-specific input is decoupled from character logic through `InputAction` assets.

---

## Core Systems & Implementation

* **Posture Mechanics:** Tracks cumulative Posture damage through `F_DamageData` structs. When the Posture threshold is exceeded, a posture-break event is triggered, applying a temporary state lock and starting a recovery timer through `Set Timer by Event`.

* **Hitbox & Collision Tracing:** Combat hit detection is handled through `SphereTraceByChannel` using a custom `CombatTrace` channel. Collision results are processed with `Break Hit Result` before applying damage through Unreal Engine's damage system.

* **Combo Chain Management:** Animation Montages use explicit section names and Animation Notify windows (`EnableComboWindow`, `ResetCombatState`) to manage combo timing, track combo progression, and enable sequential attack branches.

* **Modular Interfaces:** Uses `BPI_Attacker` and decoupled damage handling to maintain clean communication between characters, controllers, and UI systems while minimizing hard object references and unnecessary casting.

---

## Directory Structure

```text
Content/
├── Core/            # Game Modes, Player Controllers, Enums
├── Characters/      # Player Character, Animation Blueprints
│   └── Anims/       # Animation Sequences, Montages, Blend Spaces
├── UI/              # Widgets, HUD, and Stat Bindings
└── Data/            # Structs and Data Tables
```
