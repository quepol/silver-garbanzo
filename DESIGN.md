# NOVA — Game Design & Architecture Reference

## Overview
Single-file `game.html` — pure HTML5 Canvas + JS, no libraries, no build step.
First-person raycasting space shooter. Player is NOVA (a robot/AI) exploring a space station, collecting 5 crew data chips, then escaping through the exit door.

---

## Game Design

| Element | Detail |
|:--------|:-------|
| Character | **NOVA** — a robot/AI |
| Setting | Space station, sci-fi, mysterious |
| Enemies | **Alien creatures** (dark blue) + **Shadow beings** (dark purple, flickering) |
| Weapon | **Plasma blaster** — fast blue energy shots |
| Story | Find 5 missing crew data chips, then escape |
| Colors | Dark purple & blue throughout |

## Controls

| Key | Action |
|:----|:-------|
| W / ↑ | Move forward |
| S / ↓ | Move backward |
| A / ← | Rotate left |
| D / → | Rotate right |
| Space / Click | Fire plasma blaster |
| Enter | Confirm / restart |

---

## Engine Architecture (17 Sections in `<script>`)

### Section 1 — Constants & Config
- `SCREEN_W=800`, `SCREEN_H=600`, `FOV=Math.PI/3` (60°)
- `PROJ_DIST = SCREEN_W / (2 * Math.tan(FOV/2))` — perspective focal length
- `MOVE_SPEED=0.05`, `ROT_SPEED=0.03`, `MAX_DEPTH=20`
- Color palette object `COL` with all named colors
- Tile constants: `TILE_OPEN=0`, `TILE_HULL=1`, `TILE_INNER=2`, `TILE_EXIT=3`

### Section 2 — Map Data (20×20)
- Outer ring of hull walls (tile 1), interior rooms with inner walls (tile 2)
- Exit door at row 19, col 18 (tile 3)
- Player start: (1.5, 1.5) facing right (angle=0)

### Section 3 — State Machine
- States: `TITLE → PLAYING → WIN or DEAD → TITLE`
- `setState(s)` calls `initGame()` when switching to PLAYING

### Section 4 — Game World Objects
- `player` object with position, angle, health, ammo, chipsCollected
- `makeEnemy(type, x, y)` — type: `'alien'` | `'shadow'`
- `makeProjectile(x, y, dx, dy)` — with trail array
- `makeChip(x, y)` — with pulseT for animation phase
- 120 STARS for ceiling twinkle (generated once)

### Section 5 — Input Handler
- `keys{}` dict updated on keydown/keyup for smooth movement
- `e.preventDefault()` on game keys to stop page scrolling
- `fireWeapon()` called on Space keydown or canvas click (event-driven)

### Section 6 — Raycaster (DDA Algorithm)
- Per frame: compute `dirX/Y = cos/sin(angle)` and `planeX/Y` (camera plane)
- `zBuffer = new Float32Array(SCREEN_W)` — pre-allocated, reset to Infinity each frame
- **Fisheye-free perpendicular distance formula:**
  - X-side: `(mapX - player.x + (1-stepX)/2) / rayDirX`
  - Y-side: `(mapY - player.y + (1-stepY)/2) / rayDirY`
- Wall height: `lineH = Math.floor(PROJ_DIST / perpWallDist)`
- Wall color: base hue by tile type × distance falloff × side darkening (Y-side = 55%)

### Section 7 — Sprite System (Billboard Rendering)
- Gather all sprites (enemies + chips), sort back-to-front by camera-space depth
- Per sprite: project via inverse camera matrix → screen X, height
- Per column: only draw if `transformY < zBuffer[col]` (behind wall check)
- Draw via `ctx.drawImage(offscreenCanvas, srcX, 0, 1, h, col, drawStartY, 1, spriteH)`

### Section 8 — Enemy AI
- **Aliens**: straight-line chase when dist < alertRange (7 tiles), wall-slide movement
- **Shadows**: slow drift (half speed) + teleport every 280 frames to random open tile
- Both attack at range < 0.9, dealing 10 (alien) or 15 (shadow) damage per cooldown

### Section 9 — Physics & Collision
- Player: wall-slide collision with `PLAYER_RADIUS=0.28`
- Chip collection: Euclidean dist < 0.6
- Exit trigger: tile === TILE_EXIT && chipsCollected >= 5
- Ammo regen: +1 every 30 frames (2/sec)

### Section 10 — Weapon System
- Fire cooldown: 12 frames (5 shots/sec at 60fps)
- Small random spread (0.03 rad) for realistic plasma feel
- Projectile speed: 0.2 units/frame with 3-position trail

### Section 11 — World Rendering
- **Ceiling**: black fill + 120 twinkling stars
- **Floor**: dark fill + 8 horizontal + 12 converging vertical grid lines (fake perspective)
- **Walls**: column-by-column 1px `fillRect` strips from DDA output

### Section 12 — Sprite Drawing (Procedural, Off-screen Canvases)
- **Alien** (128×128): dark blue blob + lighter head ellipse + cyan eyes + 3 bezier tentacles
- **Shadow Being** (128×128): wispy bezier shape + radial gradient glow + purple eyes + flicker
- **Data Chip** (64×64): pulsing radial glow + gold diamond + bright core highlight
- **Plasma bolt**: inline glowing dot with `ctx.shadowBlur`

### Section 13 — HUD Rendering
- **Health bar** (bottom-left): color-shifts green→yellow→red
- **Ammo bar** (bottom-right): 30 individual pip rectangles
- **Crew counter** (top-center): glows gold when all 5 collected
- **Minimap** (top-right): tile colors, gold chip dots, colored enemy dots, white player triangle
- **Crosshair** (center): 4 lines with gap + center dot

### Section 14 — Screen Rendering
- **Title**: animated starfield, pulsing "NOVA" title, story text, blinking ENTER prompt
- **Win**: green "MISSION COMPLETE", flavor text
- **Dead**: red "NOVA OFFLINE", chips-collected count

### Section 15 — Audio (Web Audio API, no files)
- All sounds are procedural oscillator tones (no external assets)
- `plasma`: sawtooth 700→180 Hz | `chip`: sine ascending arpeggio
- `damage`: sawtooth 120 Hz | `death`: sawtooth 280→28 Hz
- AudioContext created lazily from first user gesture

### Section 16 — Game Loop
- `requestAnimationFrame` loop, delta-time capped to prevent huge jumps
- Order: update → computeCamera → drawCeiling → drawFloor → castRays → renderSprites → HUD → damage flash

### Section 17 — Init & Bootstrap
- `initGame()` resets player + spawns 7 enemies + places 5 chips
- Canvas 800×600, 2D context, off-screen sprite canvases pre-created

---

## Entity Positions

### Enemies
| Type   | Position    |
|--------|-------------|
| Alien  | (5.5, 3.5)  |
| Alien  | (13.5, 5.5) |
| Shadow | (9.5, 9.5)  |
| Alien  | (3.5, 14.5) |
| Shadow | (15.5, 14.5)|
| Alien  | (7.5, 16.5) |
| Shadow | (17.5, 3.5) |

### Data Chips
| Chip | Position     |
|------|--------------|
| 1    | (3.5, 1.5)   |
| 2    | (9.5, 3.5)   |
| 3    | (17.5, 7.5)  |
| 4    | (9.5, 12.5)  |
| 5    | (1.5, 17.5)  |

---

## Key Implementation Notes

1. **Fisheye**: Use perpendicular distance formula, NOT Euclidean distance
2. **zBuffer**: `Float32Array`, allocated once, `.fill(Infinity)` each frame
3. **Sprite columns**: `ctx.drawImage(sprite, srcCol, 0, 1, h, screenCol, y, 1, h)` per column
4. **AudioContext**: Create lazily from user gesture (keydown/click), not at init
5. **Canvas state**: Reset `textAlign`, `shadowBlur`, `globalAlpha` after use
6. **Hot path**: No heap allocation inside `castRays()` loop
