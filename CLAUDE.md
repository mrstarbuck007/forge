# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Forge is an open-source Magic: The Gathering rules engine and game client. It supports desktop (Java Swing), mobile (LibGDX/Android/iOS), and an Adventure mode. Java 17+ and Maven 3.8.1+ are required.

## Build Commands

```bash
# Full build (Windows/Linux)
mvn -U -B clean -P windows-linux install

# Build without tests
mvn -U -B clean -P windows-linux install -DskipTests

# Run all tests
mvn clean test

# Run a single test class
mvn test -Dtest=AbilityKeyTest

# Run checkstyle validation
mvn checkstyle:check
```

Checkstyle config is at `checkstyle.xml`. It enforces no redundant/unused imports and runs with `failOnError=true`.

## Module Architecture

The dependency chain flows strictly in one direction:

```
forge-core → forge-game → forge-ai → forge-gui → forge-gui-desktop / forge-gui-mobile
```

**forge-core** — Base utilities, card/deck/token data structures. No game rules logic.

**forge-game** — The MTG rules engine. Ability resolution, combat, mana, phases, game state. This is the heart of the project.

**forge-ai** — AI opponent decision-making. Spell/ability evaluation, combat AI, blocking logic. Depends on forge-game.

**forge-gui** — Shared UI components and all card scripting resources (`res/`). The `res/cardsfolder/` directory holds one `.txt` file per card. Also hosts networking (Netty/Jetty), deck management, and download logic.

**forge-gui-desktop** — Java Swing desktop client. Main entry point: `forge.view.Main`. JVM heap is 4 GB by default. Overlay arrows for attackers/blockers/stack targets are drawn here.

**forge-gui-mobile** — LibGDX mobile GUI shared by Android and iOS.

**forge-gui-android / forge-gui-ios** — Platform backends that delegate GUI logic to forge-gui-mobile.

**forge-gui-mobile-dev** — LibGDX desktop runner for testing mobile UI (uses LWJGL).

**adventure-editor** — Tool for authoring Adventure mode planes/quests.

## Card Scripting

Card scripts live in `forge-gui/res/cardsfolder/`, one file per card (lowercase filename, underscores for spaces, Unix line endings). Edition/set metadata is in `forge-gui/res/editions/`.

Basic structure:
```
Name:Card Name
ManaCost:2 G
Types:Creature Beast
PT:2/2
Oracle:
```

Key property prefixes: `A:` (ability), `K:` (keyword), `S:` (static), `T:` (trigger), `R:` (replacement), `SVar:` (string variable).

Conventions:
- Filename: all lowercase, skip special characters, underscores for spaces
- Unix (LF) line endings
- AI SVars go immediately before the Oracle line
- Omit parameters that match defaults (e.g., `SP$ Draw` not `SP$ Draw | Defined$ You | NumCards$ 1`)

Full scripting reference: `docs/Card-scripting-API/Card-scripting-API.md` and the wiki.

## Developer Mode (In-Game)

Enable at: Home View → Game Settings → Preferences → Gameplay Options → Developer Mode.

Useful for testing: "Setup Game State" lets you load a `.txt` file to set life totals, hands, battlefield state, and active phase directly during a game.

## Tests

- Core game logic tests: `forge-game/src/test/`
- Desktop/AI tests: `forge-gui-desktop/src/test/`
- Framework: TestNG 7.10.2

CI runs tests on Java 17 and Java 21 with a virtual framebuffer (Xvfb) for headless GUI tests.
