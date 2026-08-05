# AeroArena - Technical Architecture Document

## 1. System Overview

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              AEROARENA SYSTEM ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  ┌──────────────────────────────────┐       HTTPS/WSS        ┌──────────────────┐   │
│  │        BROWSER (CLIENT)          │ ◄─────────────────────► │  FASTAPI SERVER  │   │
│  │  ┌────────────────────────────┐  │                        │  ┌────────────┐  │   │
│  │  │      REACT APPLICATION     │  │                        │  │ REST API   │  │   │
│  │  │  ┌──────────────────────┐  │  │                        │  │ /vehicles  │  │   │
│  │  │  │   ZUSTAND STORES     │  │  │                        │  │ /players   │  │   │
│  │  │  │  • vehicles          │  │  │                        │  │ /scores    │  │   │
│  │  │  │  • player            │  │  │                        │  │ /leaderboard│  │  │
│  │  │  │  • game              │  │  │                        │  └────────────┘  │   │
│  │  │  │  • settings          │  │  │                        │  ┌────────────┐  │   │
│  │  │  └──────────────────────┘  │  │                        │  │ WEBSOCKET  │  │   │
│  │  │  ┌──────────────────────┐  │  │                        │  │ (Phase 2)  │  │   │
│  │  │  │   REACT COMPONENTS   │  │  │                        │  └────────────┘  │   │
│  │  │  │  • SelectionScreen   │  │  │                        │  ┌────────────┐  │   │
│  │  │  │  • ArenaCanvas       │  │  │                        │  │ SERVICES   │  │   │
│  │  │  │  • HUD, Menus        │  │  │                        │  │ • Auth     │  │   │
│  │  │  └──────────────────────┘  │  │                        │  │ • Stats    │  │   │
│  │  │  ┌──────────────────────┐  │  │                        │  │ • Unlocks  │  │   │
│  │  │  │  @REACT-THREE/FIBER  │  │  │                        │  └────────────┘  │   │
│  │  │  │  • SelectionScene    │  │  │                        └────────┬─────────┘   │
│  │  │  │  • ArenaScene        │  │  │                                 │           │
│  │  │  │  • PlayerVehicle     │  │  │                                 ▼           │
│  │  │  │  • ObstacleManager   │  │  │  ┌──────────────────────────────────────┐  │
│  │  │  │  • ProjectileSystem  │  │  │  │        SQLALCHEMY + SQLITE           │  │
│  │  │  │  • EffectsSystem     │  │  │  │  ┌────────────────────────────────┐  │  │
│  │  │  └──────────────────────┘  │  │  │  │  MODELS                         │  │  │
│  │  └────────────────────────────┘  │  │  │  │  • Player                     │  │  │
│  │  ┌────────────────────────────┐  │  │  │  │  • Vehicle                    │  │  │
│  │  │      THREE.JS CORE         │  │  │  │  │  • PlayerVehicle              │  │  │
│  │  │  • WebGLRenderer           │  │  │  │  │  • Score                      │  │  │
│  │  │  • Scene Graph             │  │  │  │  │  • GameSession                │  │  │
│  │  │  • AnimationMixer          │  │  │  │  └────────────────────────────────┘  │  │
│  │  │  • Raycaster               │  │  │  └──────────────────────────────────────┘  │
│  │  │  • InstancedMesh           │  │  │                                           │
│  │  └────────────────────────────┘  │  │                                           │
│  └──────────────────────────────────┘  │                                           │
│                                        │                                           │
└────────────────────────────────────────┘                                           │
                                                                                      │
```

## 2. Technology Choices & Justifications

### 2.1 Frontend Stack

| Technology | Version | Why Chosen |
|------------|---------|------------|
| **React** | 18.2+ | Component-based architecture, excellent ecosystem, concurrent features for smooth 60fps |
| **TypeScript** | 5.3+ | Type safety for game state, vehicle configs, 3D math (Vector3, Quaternion), API contracts |
| **Vite** | 5.0+ | Instant HMR, optimized production builds, native ES modules, plugin ecosystem |
| **Three.js** | r155+ | Industry standard WebGL abstraction, mature GLTF loader, massive community |
| **@react-three/fiber** | 8.15+ | Declarative Three.js, React integration, automatic cleanup, suspense support |
| **@react-three/drei** | 9.99+ | 100+ helpers (OrbitControls, Html, Effects, loaders), battle-tested |
| **Zustand** | 4.4+ | Minimal boilerplate, non-React friendly (works in game loops), middleware support |
| **Tailwind CSS** | 3.4+ | Utility-first, rapid UI dev, dark mode, tiny production bundle (JIT) |
| **GSAP** | 3.12+ | Professional animations, timeline control, scrollTrigger for UI transitions |

### 2.2 Backend Stack

| Technology | Version | Why Chosen |
|------------|---------|------------|
| **FastAPI** | 0.109+ | Async native, automatic OpenAPI/Swagger, Pydantic integration, WebSocket support |
| **Python** | 3.11+ | Familiar to user, excellent async support, rich data science ecosystem for future ML |
| **SQLAlchemy** | 2.0+ | Modern async ORM, type-safe queries, declarative models, migration support |
| **SQLite** | 3.45+ | Zero-config, file-based, perfect for prototype, WAL mode for concurrency |
| **Alembic** | 1.13+ | Schema versioning, auto-generate migrations, works with async SQLAlchemy |
| **Pydantic** | 2.6+ | Fast validation, serialization, OpenAPI schema generation |
| **Uvicorn** | 0.27+ | ASGI server, fast, production-ready, supports WebSocket |

### 2.3 3D Asset Pipeline

| Tool | Purpose | Why |
|------|---------|-----|
| **GLB/GLTF** | Model format | Web standard, binary, supports animations, materials, skins |
| **DRACO** | Geometry compression | 50-80% size reduction, native Three.js support |
| **Meshopt** | Mesh optimization | Vertex cache optimization, overdraw reduction |
| **KTX2/BasisU** | Texture compression | GPU-native, transcoding, 4-8x smaller than PNG/JPG |
| **gltf-pipeline** | CLI processing | Automated DRACO + Meshopt + texture compression |
| **Blender** | Authoring | Free, open-source, GLTF export with animation support |

## 3. Detailed Component Architecture

### 3.1 Frontend State Management (Zustand Stores)

```
┌─────────────────────────────────────────────────────────────────┐
│                        ZUSTAND STORE ARCHITECTURE               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  useVehicleStore                                                │
│  ├─ vehicles: Vehicle[]           ← Fetched from /api/vehicles │
│  ├─ selectedVehicle: VehicleKey   ← User selection             │
│  ├─ fetchVehicles()               ← API call                   │
│  ├─ selectVehicle(key)            ← Updates selection          │
│  └─ getVehicle(key): Vehicle      ← Lookup helper              │
│                                                                 │
│  usePlayerStore                                                 │
│  ├─ player: PlayerProfile | null  ← Current player             │
│  ├─ unlockedVehicles: VehicleKey[]← Owned vehicles             │
│  ├─ highScores: Score[]           ← Personal bests             │
│  ├─ createPlayer(username)        ← POST /api/players          │
│  ├─ unlockVehicle(vehicleId)      ← POST unlock endpoint       │
│  └─ fetchProfile()                ← GET /api/players/:id       │
│                                                                 │
│  useGameStore (transient, reset per session)                    │
│  ├─ status: 'idle' | 'playing' | 'paused' | 'gameover'         │
│  ├─ score: number                 ← Current run score          │
│  ├─ combo: number                 ← Destruction multiplier     │
│  ├─ health: number                ← Vehicle HP                 │
│  ├─ ammo: { primary: ∞, secondary: number }                   │
│  ├─ distance: number              ← Meters traveled            │
│  ├─ obstaclesDestroyed: number    ← Count                      │
│  ├─ startGame(vehicle)            ← Initialize session         │
│  ├─ addScore(points)              ← With combo logic           │
│  ├─ takeDamage(amount)            ← Health management          │
│  ├─ pause() / resume()            ← State transitions          │
│  └─ endGame()                     ← Submit score, cleanup      │
│                                                                 │
│  useSettingsStore (persisted to localStorage + SQLite)          │
│  ├─ graphics: 'low' | 'medium' | 'high'                        │
│  ├─ controls: ControlScheme                                     │
│  ├─ audio: { master: 0.7, sfx: 0.8, music: 0.5 }               │
│  ├─ reducedMotion: boolean                                      │
│  └─ save() / load()                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Three.js Scene Graph (ArenaScene)

```
ArenaScene (Canvas)
├── PerspectiveCamera (follows PlayerVehicle)
├── Scene
│   ├── ArenaEnvironment
│   │   ├── Skybox (CubeTextureLoader, 6 faces)
│   │   ├── GroundPlane (PlaneGeometry + MeshStandardMaterial)
│   │   ├── HemisphereLight (sky + ground color)
│   │   ├── DirectionalLight (sun, casts shadows)
│   │   └── Fog (exponential, distance-based)
│   │
│   ├── PlayerVehicle (Group)
│   │   ├── Body (GLTF mesh, animated)
│   │   ├── Propeller/Rotor (separate mesh, animated via AnimationMixer)
│   │   ├── WeaponMuzzle (Object3D - projectile spawn point)
│   │   ├── CameraPivot (Object3D - camera follow target)
│   │   ├── CollisionHull (invisible Mesh, simplified geometry)
│   │   └── TrailRenderer (Line2 / MeshLine for contrails)
│   │
│   ├── ObstacleManager (Group)
│   │   ├── StaticObstacles (InstancedMesh - buildings, mountains)
│   │   │   ├── geometry: BoxGeometry / CylinderGeometry
│   │   │   ├── material: MeshStandardMaterial (shared)
│   │   │   └── instanceMatrix: Float32Array (position/rotation/scale)
│   │   ├── DynamicObstacles (Array<ObstacleEntity>)
│   │   │   ├── EnemyDrone (animated, simple AI)
│   │   │   ├── FallingDebris (physics-based)
│   │   │   └── TargetMarker (glowing, destructible, scores points)
│   │   └── SpatialIndex (UniformGrid for broad-phase culling)
│   │
│   ├── ProjectileSystem (Group)
│   │   ├── HitscanProjectiles (Raycaster results, instant)
│   │   │   ├── MuzzleFlash (Sprite, additive blending)
│   │   │   └── ImpactEffect (ParticleSystem at hit point)
│   │   ├── PhysicalProjectiles (Mesh pool - missiles, curve shots)
│   │   │   ├── Missile (homing logic, trail)
│   │   │   ├── CurveBall (Bezier path, physics)
│   │   │   └── LaserBeam (Line2, animated)
│   │   └── ObjectPool (pre-allocated meshes, reuse)
│   │
│   └── EffectsSystem (Group)
│       ├── ExplosionPool (ParticleSystem, ExpandingRing, Debris)
│       ├── ScreenShake (camera offset, intensity decay)
│       ├── DamageNumbers (HTML overlay via <Html> from drei)
│       └── PostProcessing (EffectComposer)
│           ├── RenderPass
│           ├── UnrealBloomPass (bloom threshold/strength/radius)
│           ├── FXAAPass (anti-aliasing)
│           └── ShaderPass (custom: vignette, chromatic aberration)
│
└── OrbitControls (disabled during gameplay, enabled in preview)
```

### 3.3 Backend Module Structure

```
backend/app/
├── main.py                 # FastAPI app, lifespan, middleware, routers
├── config.py               # Pydantic Settings (env vars, defaults)
├── database.py             # AsyncEngine, AsyncSessionLocal, Base
├── models/                 # SQLAlchemy Declarative Models
│   ├── __init__.py
│   ├── player.py           # Player, PlayerVehicle
│   ├── vehicle.py          # Vehicle (catalog)
│   ├── score.py            # Score, GameSession
│   └── mixins.py           # TimestampMixin, IDMixin
├── schemas/                # Pydantic Request/Response Models
│   ├── __init__.py
│   ├── vehicle.py          # VehicleRead, VehicleList, VehicleStats
│   ├── player.py           # PlayerCreate, PlayerRead, PlayerProfile
│   ├── score.py            # ScoreCreate, ScoreRead, LeaderboardEntry
│   └── common.py           # PaginatedResponse, ErrorResponse
├── api/                    # API Route Handlers
│   ├── __init__.py
│   ├── vehicles.py         # GET /vehicles, GET /vehicles/{key}
│   ├── players.py          # POST /players, GET /players/{id}, unlocks
│   ├── scores.py           # POST /scores, GET /leaderboard
│   └── websocket.py        # Phase 2: WebSocket endpoints
├── services/               # Business Logic (independent of HTTP)
│   ├── __init__.py
│   ├── vehicle_service.py  # Catalog queries, unlock validation
│   ├── player_service.py   # Profile management, progression
│   ├── score_service.py    # Scoring calculations, leaderboards
│   └── session_service.py  # Game session lifecycle
├── seed.py                 # Initial vehicle data population
└── utils/                  # Helpers
    ├── __init__.py
    ├── physics.py          # Shared physics constants
    └── validators.py       # Custom Pydantic validators
```

### 3.4 Database Schema (ER Diagram)

```
┌─────────────────┐       ┌──────────────────────┐       ┌─────────────────┐
│     PLAYER      │       │   PLAYER_VEHICLE     │       │    VEHICLE      │
├─────────────────┤       ├──────────────────────┤       ├─────────────────┤
│ id (PK)         │◄──────│ id (PK)              │       │ id (PK)         │
│ username (UK)   │       │ player_id (FK)       │──────►│ key (UK)        │
│ created_at      │       │ vehicle_id (FK)      │       │ name            │
│ total_score     │       │ unlocked_at          │       │ category        │
│ total_playtime  │       │ upgrades (JSON)      │       │ stats (JSON)    │
│ settings (JSON) │       └──────────────────────┘       │ weapon (JSON)   │
└────────┬────────┘                                      │ special (JSON)  │
         │                                               │ model_path      │
         │                        ┌──────────────────┐   │ unlock_score    │
         │                        │      SCORE       │   │ is_default      │
         └───────────────────────►│ id (PK)          │   │ animation_cfg   │
                                  │ player_id (FK)   │   └─────────────────┘
                                  │ vehicle_id (FK)  │
                                  │ score            │
                                  │ duration         │
                                  │ obstacles_destroyed
                                  │ distance_traveled
                                  │ created_at
                                  └────────┬─────────┘
                                           │
                                  ┌────────┴────────┐
                                  │  GAME_SESSION   │
                                  ├─────────────────┤
                                  │ id (PK)         │
                                  │ player_id (FK)  │
                                  │ vehicle_id (FK) │
                                  │ started_at      │
                                  │ ended_at        │
                                  │ final_score     │
                                  │ metadata (JSON) │
                                  └─────────────────┘
```

## 4. Data Flow Sequences

### 4.1 Initial Load & Vehicle Selection

```
User                    Browser (React)                    FastAPI Server           SQLite
  │                          │                                 │                       │
  │                          │  GET /api/vehicles              │                       │
  │                          │ ─────────────────────────────►  │                       │
  │                          │                                 │  SELECT * FROM vehicles
  │                          │                                 │  ──────────────────►
  │                          │                                 │                       │
  │                          │  [Vehicle[]]                    │                       │
  │                          │ ◄─────────────────────────────  │                       │
  │                          │                                 │                       │
  │  Renders VehicleGrid     │                                 │                       │
  │  User clicks F-16        │                                 │                       │
  │                          │  useVehicleStore.select('f16')  │                       │
  │                          │                                 │                       │
  │                          │  <SelectionScene> loads         │                       │
  │                          │  useGLTF('/models/f16.glb')     │                       │
  │                          │  (cached by drei)               │                       │
  │                          │                                 │                       │
  │  3D Preview renders      │                                 │                       │
  │  StatsPanel shows stats  │                                 │                       │
  │                          │                                 │                       │
  │  Clicks "PLAY"           │                                 │                       │
```

### 4.2 Gameplay Loop (Client-Side Only During Play)

```
requestAnimationFrame (60Hz target)
        │
        ▼
┌───────────────────┐
│  Input Reading    │  ◄─── Keyboard/Gamepad state (Zustand/Ref)
│  (useKeyboard)    │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  Physics Step     │  Fixed timestep (1/60s)
│  (useGameLoop)    │  ├─ Apply forces (thrust, drag, lift, gravity)
│                   │  ├─ Integrate velocity → position
│                   │  ├─ Apply angular velocity → rotation
│                   │  └─ Clamp boundaries
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  Collision Check  │  Broad: Spatial grid query
│                   │  Narrow: AABB / Raycast / Sphere sweep
│                   │  Response: Damage, destroy, score
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  Weapon Update    │  ├─ Cooldown timers
│                   │  ├─ Projectile physics (pool)
│                   │  └─ Hit detection (raycast vs obstacles)
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  Camera Update    │  Smooth follow (lerp)
│                   │  Offset: behind + above vehicle
│                   │  LookAt: vehicle + velocity lead
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  Render (Three.js)│  ├─ Scene.traverse(animate)
│                   │  ├─ AnimationMixer.update(dt)
│                   │  ├─ EffectComposer.render()
│                   │  └─ Post-processing passes
└─────────┬─────────┘
          │
          ▼
    (Next Frame)
```

### 4.3 Session End & Score Submission

```
Game Over                Browser                         FastAPI                      SQLite
  │                        │                                │                          │
  │                        │  POST /api/scores              │                          │
  │                        │  { vehicle_key, score,        │                          │
  │                        │    duration, obstacles,       │                          │
  │                        │    distance }                 │                          │
  │                        │ ────────────────────────────►  │                          │
  │                        │                                │  INSERT INTO scores      │
  │                        │                                │  UPDATE player SET       │
  │                        │                                │    total_score = ...     │
  │                        │                                │  CHECK unlock conditions │
  │                        │                                │    INSERT INTO player_   │
  │                        │                                │    vehicle IF met        │
  │                        │                                │                          │
  │                        │  { score_id, new_unlocks,     │                          │
  │                        │   updated_profile }           │                          │
  │                        │ ◄────────────────────────────  │                          │
  │                        │                                │                          │
  │  Shows GameOverScreen  │                                │                          │
  │  with new unlocks      │                                │                          │
```

## 5. Performance Budget & Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Initial Load** | < 3s (3G) | Lighthouse / WebPageTest |
| **JS Bundle (gz)** | < 500 KB | `vite build` + `gzip-size` |
| **Model Load (per vehicle)** | < 500 KB | DRACO-compressed GLB |
| **Frame Time** | < 16.67ms (60fps) | Chrome DevTools Performance |
| **Memory (heap)** | < 100 MB | Chrome DevTools Memory |
| **Draw Calls** | < 100 | Three.js `renderer.info.render.calls` |
| **Triangles/Frame** | < 100k | Three.js `renderer.info.render.triangles` |
| **API Latency (local)** | < 50ms | FastAPI `/health` endpoint |

## 6. Asset Requirements

### 6.1 Vehicle Models (GLB Specification)

```
Required Structure per Vehicle:
VehicleRoot (Group)
├── Body (Mesh)
│   ├── geometry: BufferGeometry (indexed, UV unwrapped)
│   ├── material: MeshStandardMaterial
│   │   ├── map: KTX2 texture (albedo)
│   │   ├── normalMap: KTX2
│   │   ├── roughnessMap: KTX2
│   │   └── metalnessMap: KTX2
│   └── morphTargets: optional (damage states)
│
├── Propeller/Rotor (Mesh, separate for animation)
│   ├── name: "propeller" / "main_rotor" / "tail_rotor"
│   └── animation: "spin" (loop, 1s duration)
│
├── WeaponMuzzle (Object3D)
│   ├── name: "muzzle_primary" / "muzzle_secondary"
│   └── position: tip of gun/launcher
│
├── CameraPivot (Object3D)
│   ├── name: "camera_pivot"
│   └── position: behind vehicle, slightly up
│
└── CollisionHull (Mesh, invisible)
    ├── name: "collision_hull"
    ├── geometry: ConvexHull / Box / Capsule (simplified)
    └── visible: false
    └── userData: { collisionType: 'vehicle' }

Animation Clips (named):
- "idle" (loop, subtle breathing)
- "propeller_spin" / "rotor_spin" (loop, 0.5-1s)
- "boost" (once, 1s) - for afterburner
- "damage" (once, 0.5s) - hit reaction
```

### 6.2 Compression Pipeline

```bash
# 1. Export from Blender: File → Export → glTF 2.0 (.glb)
#    ✓ Include: Selected Objects, Tangents, UVs, Normals
#    ✓ Animation: Always Sample Animations, NLA Strips

# 2. Compress geometry (DRACO + Meshopt)
npx gltf-transform draco input.glb output.glb \
  --level=7 --quantize-position=14 --quantize-normal=10 --quantize-texcoord=12

npx gltf-transform meshopt input.glb output.glb

# 3. Compress textures (to KTX2/UASTC)
npx gltf-transform ktx2 input.glb output.glb \
  --uastc --zstd --generate-mipmaps

# 4. Verify
npx gltf-transform inspect output.glb
# Should show: DRACO, Meshopt, KTX2, < 500KB total
```

## 7. Security Considerations

| Area | Measure |
|------|---------|
| **Input Validation** | Pydantic schemas on all endpoints, strict types |
| **SQL Injection** | SQLAlchemy ORM (parameterized queries), no raw SQL |
| **XSS** | React auto-escapes, no `dangerouslySetInnerHTML` |
| **CORS** | Restricted to frontend origin in production |
| **Rate Limiting** | SlowAPI / Redis (future) for score submission |
| **Data Exposure** | Minimal fields in API responses, no internal IDs leaked |
| **WebSocket Auth** | Token-based (Phase 2) |

## 8. Testing Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                        TESTING PYRAMID                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────┐                                          │
│   │   E2E (Playwright)      │  5-10 tests                       │
│   │   • Full game flow      │  • Selection → Play → GameOver   │
│   │   • Score submission    │  • Leaderboard persistence       │
│   │   • Settings persist    │  • Mobile viewport               │
│   └────────┬────────┘                                          │
│            │                                                   │
│   ┌────────┴────────┐                                          │
│   │ Integration     │  20-30 tests                             │
│   │ (pytest + httpx)│  • API endpoints (all CRUD)              │
│   │                 │  • Database operations                   │
│   │                 │  • Vehicle unlock logic                  │
│   │                 │  • Scoring calculations                  │
│   └────────┬────────┘                                          │
│            │                                                   │
│   ┌────────┴────────┐                                          │
│   │ Unit (Vitest)   │  50+ tests                               │
│   │                 │  • Physics: velocity, drag, clamping    │
│   │                 │  • Scoring: combos, multipliers         │
│   │                 │  • Vehicle configs: validation          │
│   │                 │  • Store actions: select, unlock, etc.  │
│   │                 │  • Utils: math helpers, formatters      │
│   └─────────────────┘                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 9. Deployment Architecture (Production)

```
┌─────────────────────────────────────────────────────────────────┐
│                        PRODUCTION DEPLOYMENT                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐  │
│  │   CDN/EDGE   │      │   FRONTEND   │      │   BACKEND    │  │
│  │  (Cloudflare │      │  (Vercel/    │      │  (Railway/   │  │
│  │   / Vercel)  │      │   Netlify)   │      │   Render/    │  │
│  └──────┬───────┘      └──────┬───────┘      │   Fly.io)    │  │
│         │                     │              └──────┬───────┘  │
│         │                     │                     │          │
│         ▼                     ▼                     ▼          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    USER BROWSER                           │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Frontend: Static assets (JS, CSS, GLB, KTX2) → CDN            │
│  Backend:  Docker container → Railway/Render/Fly.io            │
│  Database: SQLite file → Persistent volume (Railway/Render)    │
│  Assets:   Large models → CDN (or S3 + CloudFront)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 10. File Structure (Complete)

```
aeroplanegameapp/
├── README.md
├── docs/
│   ├── architecture.md          # This file
│   ├── techstack.md             # Technology decisions summary
│   ├── API.md                   # OpenAPI spec + examples
│   ├── ASSET_PIPELINE.md        # Blender → Web workflow
│   └── DEPLOYMENT.md            # Production deployment guide
│
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── player.py
│   │   │   ├── vehicle.py
│   │   │   ├── score.py
│   │   │   └── mixins.py
│   │   ├── schemas/
│   │   │   ├── __init__.py
│   │   │   ├── vehicle.py
│   │   │   ├── player.py
│   │   │   ├── score.py
│   │   │   └── common.py
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── vehicles.py
│   │   │   ├── players.py
│   │   │   ├── scores.py
│   │   │   └── websocket.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── vehicle_service.py
│   │   │   ├── player_service.py
│   │   │   ├── score_service.py
│   │   │   └── session_service.py
│   │   ├── utils/
│   │   │   ├── __init__.py
│   │   │   ├── physics.py
│   │   │   └── validators.py
│   │   └── seed.py
│   ├── alembic/
│   │   ├── env.py
│   │   ├── script.py.mako
│   │   └── versions/
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── conftest.py
│   │   ├── test_vehicles.py
│   │   ├── test_players.py
│   │   ├── test_scores.py
│   │   └── test_services.py
│   ├── requirements.txt
│   ├── pyproject.toml
│   ├── .env.example
│   └── README.md
│
├── frontend/
│   ├── public/
│   │   ├── models/              # GLB files (or use CDN URLs)
│   │   ├── textures/
│   │   ├── audio/
│   │   └── favicon.ico
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── vite-env.d.ts
│   │   │
│   │   ├── stores/
│   │   │   ├── useVehicleStore.ts
│   │   │   ├── usePlayerStore.ts
│   │   │   ├── useGameStore.ts
│   │   │   └── useSettingsStore.ts
│   │   │
│   │   ├── components/
│   │   │   ├── ui/
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Card.tsx
│   │   │   │   ├── ProgressBar.tsx
│   │   │   │   ├── Modal.tsx
│   │   │   │   └── StatBar.tsx
│   │   │   ├── selection/
│   │   │   │   ├── VehicleGrid.tsx
│   │   │   │   ├── VehicleCard.tsx
│   │   │   │   ├── VehiclePreview.tsx
│   │   │   │   ├── StatsPanel.tsx
│   │   │   │   └── SelectionScreen.tsx
│   │   │   ├── arena/
│   │   │   │   ├── ArenaCanvas.tsx
│   │   │   │   ├── HUD.tsx
│   │   │   │   ├── PauseMenu.tsx
│   │   │   │   ├── GameOverScreen.tsx
│   │   │   │   └── ControlsHelp.tsx
│   │   │   └── common/
│   │   │       ├── LoadingScreen.tsx
│   │   │       └── ErrorBoundary.tsx
│   │   │
│   │   ├── scenes/
│   │   │   ├── SelectionScene.tsx
│   │   │   └── ArenaScene.tsx
│   │   │       ├── ArenaEnvironment.tsx
│   │   │       ├── PlayerVehicle.tsx
│   │   │       ├── ObstacleManager.tsx
│   │   │       ├── ProjectileSystem.tsx
│   │   │       ├── CollisionSystem.tsx
│   │   │       └── EffectsSystem.tsx
│   │   │
│   │   ├── hooks/
│   │   │   ├── useGameLoop.ts
│   │   │   ├── useKeyboardControls.ts
│   │   │   ├── useGamepadControls.ts
│   │   │   ├── useAssetPreload.ts
│   │   │   └── useLocalStorage.ts
│   │   │
│   │   ├── systems/
│   │   │   ├── physics.ts
│   │   │   ├── input.ts
│   │   │   ├── scoring.ts
│   │   │   └── vehicleConfigs.ts
│   │   │
│   │   ├── types/
│   │   │   ├── vehicle.ts
│   │   │   ├── player.ts
│   │   │   ├── game.ts
│   │   │   └── api.ts
│   │   │
│   │   ├── services/
│   │   │   ├── api.ts
│   │   │   ├── vehicleService.ts
│   │   │   ├── playerService.ts
│   │   │   └── scoreService.ts
│   │   │
│   │   ├── assets/
│   │   │   └── (shaders, generated types)
│   │   │
│   │   └── styles/
│   │       ├── globals.css
│   │       └── tailwind.css
│   │
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   ├── tsconfig.node.json
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── .eslintrc.json
│   ├── .prettierrc
│   └── README.md
│
├── docker-compose.yml           # Local development stack
├── .gitignore
└── LICENSE
```

## 11. Development Workflow

### 11.1 Local Development

```bash
# Terminal 1: Backend
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
alembic upgrade head
python -m app.seed
uvicorn app.main:app --reload --port 8000

# Terminal 2: Frontend
cd frontend
npm install
npm run dev
# Opens http://localhost:5173
```

### 11.2 Code Quality Gates

```bash
# Frontend
npm run lint          # ESLint
npm run typecheck     # tsc --noEmit
npm run test          # Vitest
npm run test:e2e      # Playwright

# Backend
ruff check .          # Linting (fast)
mypy app/             # Type checking
pytest                # Unit + Integration
```

### 11.3 Git Hooks (Husky)

```json
// .husky/pre-commit
#!/bin/sh
cd frontend && npm run lint && npm run typecheck
cd ../backend && ruff check . && mypy app/
```

## 12. Future Extensibility Points

| Feature | Extension Point | Effort |
|---------|----------------|--------|
| **Multiplayer** | WebSocket server + state sync | Medium |
| **New Vehicles** | Add GLB + config + seed entry | Low |
| **New Arenas** | New Environment component + spawn config | Low |
| **Game Modes** | GameStore status enum + mode logic | Medium |
| **Mobile** | Touch controls hook + responsive UI | Medium |
| **AI Opponents** | ObstacleManager → EntityManager + behavior trees | High |
| **Replay System** | Serialize inputs → deterministic replay | Medium |
| **Modding** | External vehicle configs (JSON) + asset loading | Medium |

---

*Document Version: 1.0*  
*Last Updated: 2026-08-06*  
*Status: Planning Phase - Ready for Implementation*