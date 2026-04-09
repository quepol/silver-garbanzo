# NOVA — First-Person Space Shooter

**Build one file: `game.html` — pure HTML5 Canvas \+ JS, no libraries, no build step.**

## Game Design

| Element | Detail |
| :---- | :---- |
| Character | **NOVA** — a robot/AI |
| Setting | Space station, sci-fi, mysterious |
| Enemies | **Alien creatures** (dark blue) \+ **Shadow beings** (dark purple, flickering) |
| Weapon | **Plasma blaster** — fast blue energy shots |
| Story | Find 5 missing crew data chips, then escape |
| Colors | Dark purple & blue throughout |

## Engine

- **Raycasting** (Wolfenstein/DOOM style) — DDA algorithm, one ray per screen column  
- Billboard sprites for enemies & collectibles with z-buffer depth testing  
- 20×20 tile grid map (0=open, 1=hull wall, 2=inner wall, 3=exit door)

## Controls

- `W/S` or `↑↓` — move forward/back  
- `A/D` or `←→` — rotate left/right  
- `Space` or click — fire plasma blaster

## HUD

- Health bar (bottom left), ammo bar (bottom right, regenerates)  
- Crew counter top center: `CREW DATA: 0/5`  
- Minimap top right (shows walls, enemies, chips, player)  
- Crosshair center screen

## Game States

1. **Title screen** — NOVA logo, story text, press ENTER  
2. **Playing** — main loop  
3. **Win** — "MISSION COMPLETE" after reaching exit with all 5 chips  
4. **Dead** — "NOVA OFFLINE" with restart

## Enemies

- **Aliens** (dark blue): chase player when within range, 2 hits to kill  
- **Shadow beings** (purple): slow drift \+ occasional teleport, 3 hits, flicker visually

## Visual Style

- Ceiling: black with twinkling stars  
- Floor: dark with grid lines (metal grating)  
- Walls: purple/blue, distance shading (closer \= brighter)  
- Chips: glowing gold diamond shape, pulsing  
- Aliens: blob body, cyan eyes, tentacles  
- Shadow beings: wispy bezier shape, purple glowing eyes, frame-based flicker.
