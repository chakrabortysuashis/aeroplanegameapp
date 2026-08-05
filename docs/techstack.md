# AeroArena - Technology Stack Decisions

## Executive Summary

This document captures the key technology decisions for the AeroArena 3D flying game prototype, with justifications for each choice.

---

## Frontend Technologies

### Core Framework: React 18 + TypeScript + Vite

| Aspect | Decision | Rationale |
|--------|----------|-----------|
| **UI Framework** | React 18 | Component composition, hooks, concurrent rendering for 60fps, massive ecosystem |
| **Language** | TypeScript 5.3+ | Essential for 3D math types (Vector3, Quaternion, Matrix4), game state, API contracts |
| **Build Tool** | Vite 5+ | Instant HMR, optimized Rollup builds, native ESM, excellent plugin ecosystem |
| **Package Manager** | npm (or pnpm) | Standard, good monorepo support if needed later |

### 3D Rendering: Three.js + React Three Fiber

| Layer | Library | Version | Why |
|-------|---------|---------|-----|
| **Core 3D** | Three.js | r155+ | WebGL standard, mature, excellent GLTF support, active maintenance |
| **React Bridge** | @react-three/fiber | 8.15+ | Declarative, automatic cleanup, suspense, event system, <Canvas> component |
| **Helpers** | @react-three/drei | 9.99+ | 100+ production-ready helpers: OrbitControls, Html, Effects, loaders, Shapes |
| **Post-Processing** | @react-three/postprocessing | 2.15+ | EffectComposer integration, Bloom, FXAA, custom shaders |
| **Animation** | @react-three/fiber + GSAP | 3.12+ | Spring physics, timelines, scroll-based UI animations |

**Alternative Considered**: Babylon.js + React Babylon
- *Rejected*: Smaller ecosystem, less community examples, steeper learning curve for React devs

**Alternative Considered**: Plain Three.js (no React bridge)
- *Rejected*: Manual lifecycle management, imperative code harder to maintain, no React state integration

### State Management: Zustand

| Feature | Benefit |
|---------|---------|
| **Minimal API** | `create<State>()((set) => ({ ... }))` - no providers, no context |
| **Non-React Friendly** | Works in game loops, physics systems, Three.js event handlers |
| **Middleware** | `persist` for settings, `devtools` for debugging, `immer` for immutable updates |
| **Performance** | Selective subscriptions, no re-renders on unrelated changes |
| **Bundle Size** | ~1KB gzipped |

**Alternatives Considered**:
- **Redux Toolkit**: Overkill for prototype, more boilerplate
- **Jotai/Recoil**: Atomic model not needed, extra complexity
- **React Context**: Performance issues with high-frequency updates (game loop)

### Styling: Tailwind CSS

| Benefit | Detail |
|---------|--------|
| **JIT Compiler** | Only used styles in production bundle (~10KB) |
| **Utility-First** | Rapid UI development, no CSS files to maintain |
| **Dark Mode** | Built-in `dark:` variant, CSS variables support |
| **Responsive** | Mobile-first breakpoints, container queries |
| **Design System** | Consistent spacing, colors, typography via config |

### Input Handling

| Method | Implementation |
|--------|----------------|
| **Keyboard** | Custom `useKeyboardControls` hook with `keydown`/`keyup` listeners |
| **Gamepad** | `useGamepadControls` hook using Gamepad API polling |
| **Pointer Lock** | For mouse aim (future), `requestPointerLock()` |
| **Touch** | Virtual joystick component (Phase 2) |

### Asset Loading

| Tool | Purpose |
|------|---------|
| **useGLTF** | drei hook, DRACO/KTX2 support, caching, progress events |
| **useTexture** | KTX2/BasisU, automatic mipmaps, anisotropy |
| **gltf-transform** | CLI for DRACO + Meshopt + KTX2 pipeline |

---

## Backend Technologies

### API Framework: FastAPI

| Feature | Benefit |
|---------|---------|
| **Async Native** | `async def` endpoints, handles 10k+ concurrent connections |
| **Auto Docs** | Swagger UI at `/docs`, ReDoc at `/redoc` - always in sync |
| **Pydantic Integration** | Request/response validation, serialization, OpenAPI schemas |
| **WebSocket Support** | Native `WebSocket` class, dependency injection |
| **Dependency Injection** | `Depends()` for database sessions, auth, pagination |
| **Performance** | Starlette + Uvicorn, one of fastest Python frameworks |

### Database: SQLite + SQLAlchemy 2.0 (Async)

| Aspect | Decision |
|--------|----------|
| **Engine** | SQLite (file-based, zero-config, WAL mode for concurrency) |
| **ORM** | SQLAlchemy 2.0 async (`AsyncSession`, `async_sessionmaker`) |
| **Migrations** | Alembic with async support |
| **Models** | Declarative with `Mapped` type annotations |

**Why Not PostgreSQL/MySQL?**
- Prototype scope: single-user, local development
- SQLite handles 100k+ rows easily
- Zero infrastructure, easy distribution
- Can migrate to PostgreSQL later with minimal changes (SQLAlchemy dialect swap)

### Validation: Pydantic v2

| Feature | Use Case |
|---------|----------|
| **Strict Types** | `VehicleKey = Literal['f16', 'apache', 'soccer_ball']` |
| **Computed Fields** | `total_score: int = Field(ge=0)` |
| **Serialization** | `model_dump(mode='json')` for API responses |
| **Custom Validators** | `@field_validator('unlock_score')` for business rules |

### Server: Uvicorn

- ASGI server, production-ready
- Workers for CPU-bound tasks (not needed for this prototype)
- `--reload` for development

---

## 3D Asset Pipeline

### Model Format: GLB (Binary GLTF)

| Requirement | Specification |
|-------------|---------------|
| **Format** | GLB (single file, binary) |
| **Version** | GLTF 2.0 |
| **Compression** | DRACO (geometry) + Meshopt (vertex cache) |
| **Textures** | KTX2 / Basis Universal (UASTC + ZSTD) |
| **Max Size** | 500KB per vehicle (compressed) |
| **Animations** | Named clips, skeletal or morph targets |

### Asset Sources (Prototype)

| Vehicle | Source | License |
|---------|--------|---------|
| F-16 Falcon | Kenney.nl / Sketchfab (free) | CC0 / CC-BY |
| AH-64 Apache | Kenney.nl / Custom Blender | CC0 |
| Soccer Ball | Custom (simple geometry) | Original |
| MQ-9 Reaper | Sketchfab free | CC-BY |
| Jetpack Ranger | Custom Blender | Original |

### Compression Pipeline

```bash
# Install tools
npm install -g @gltf-transform/cli

# 1. DRACO geometry compression (level 7 = max)
gltf-transform draco input.glb output.glb \
  --level=7 \
  --quantize-position=14 \
  --quantize-normal=10 \
  --quantize-texcoord=12

# 2. Meshopt optimization
gltf-transform meshopt output.glb output.glb

# 3. KTX2 texture compression (UASTC + ZSTD)
gltf-transform ktx2 output.glb output.glb \
  --uastc --zstd --generate-mipmaps

# 4. Verify
gltf-transform inspect output.glb
```

### Texture Guidelines

| Map Type | Format | Resolution | Notes |
|----------|--------|------------|-------|
| Albedo (Base Color) | KTX2/UASTC | 1024x1024 | sRGB |
| Normal | KTX2/UASTC | 1024x1024 | Linear |
| Roughness/Metalness | KTX2/UASTC | 512x512 | Packed in R/G channels |
| Emissive | KTX2/UASTC | 512x512 | For glowing parts |

---

## Development Tools

### Code Quality

| Tool | Config | Purpose |
|------|--------|---------|
| **ESLint** | `eslint.config.js` | Linting, React hooks rules, TypeScript |
| **Prettier** | `.prettierrc` | Formatting, single quotes, trailing commas |
| **TypeScript** | `tsconfig.json` | Strict mode, no implicit any |
| **Ruff** | `pyproject.toml` | Python linting (100x faster than flake8) |
| **MyPy** | `pyproject.toml` | Python static typing |

### Testing

| Layer | Tool | Scope |
|-------|------|-------|
| **Unit (Frontend)** | Vitest | Physics, scoring, stores, utils |
| **Unit (Backend)** | pytest | Services, validators, schemas |
| **Integration (Backend)** | pytest + httpx | API endpoints, database |
| **E2E** | Playwright | Full user flows, visual regression |

### Git Hooks

```bash
# Husky + lint-staged
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

```json
// package.json
"lint-staged": {
  "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
  "*.py": ["ruff check --fix", "ruff format"]
}
```

---

## Performance Budget

| Metric | Budget | Tool |
|--------|--------|------|
| **Initial JS (gz)** | < 500 KB | `vite build && gzip-size dist/assets/*.js` |
| **Three.js Core** | ~120 KB | Included in bundle |
| **React + Vendor** | ~150 KB | React, Zustand, GSAP |
| **Game Code** | ~80 KB | Components, systems, hooks |
| **Models (per vehicle)** | < 500 KB | DRACO + KTX2 compressed |
| **Textures (per vehicle)** | < 200 KB | KTX2/UASTC |
| **Frame Time** | < 16.67ms | Chrome DevTools Performance |
| **Draw Calls** | < 100 | `renderer.info.render.calls` |
| **Triangles** | < 100k/frame | `renderer.info.render.triangles` |
| **Memory** | < 100 MB | Chrome DevTools Memory |

---

## Browser Support

| Browser | Minimum Version | Notes |
|---------|-----------------|-------|
| **Chrome** | 90+ | Full WebGL2, Pointer Lock, Gamepad |
| **Firefox** | 88+ | Full support |
| **Safari** | 15+ | WebGL2, some WebGPU prep |
| **Edge** | 90+ | Chromium-based, same as Chrome |

**Polyfills Needed**: None (modern APIs only)

**Fallbacks**:
- `prefers-reduced-motion` → disable animations
- No WebGL2 → Show unsupported message (rare)

---

## Deployment Targets

### Frontend: Vercel / Netlify / Cloudflare Pages

- Static asset hosting + CDN
- Automatic HTTPS
- Edge functions for API proxy (if needed)
- Preview deployments per PR

### Backend: Railway / Render / Fly.io

- Docker container deployment
- Persistent volume for SQLite
- Automatic HTTPS
- Custom domains
- Free tiers sufficient for prototype

### Database: SQLite on Persistent Volume

- Single file: `data/aeroarena.db`
- WAL mode enabled for read concurrency
- Backup: copy file (simple)

---

## Decision Log

| Date | Decision | Context | Alternatives Rejected |
|------|----------|---------|----------------------|
| 2026-08-06 | React + Three.js (R3F) | 3D web game prototype | Babylon.js, plain Three.js |
| 2026-08-06 | Zustand for state | Game loop needs non-React store | Redux, Jotai, Context |
| 2026-08-06 | FastAPI + SQLite | Python backend, zero-config DB | Flask + PostgreSQL, Node.js |
| 2026-08-06 | GLB + DRACO + KTX2 | Web standard, compressed | FBX, OBJ, uncompressed GLTF |
| 2026-08-06 | Vite + Tailwind | Fast dev, small bundle | Webpack, CSS Modules |
| 2026-08-06 | Fixed timestep physics | Deterministic, 60Hz | Variable timestep |

---

## Future Technology Considerations (Post-Prototype)

| Feature | Technology | Timeline |
|---------|------------|----------|
| **Multiplayer** | WebSocket + Yjs / Colyseus | Phase 2 |
| **WebGPU** | Three.js WebGPURenderer | When stable |
| **Physics Engine** | Rapier (WASM) / Cannon.js | If arcade physics insufficient |
| **Global Leaderboard** | PostgreSQL + Redis | Phase 2 |
| **Mobile PWA** | Workbox + Manifest | Phase 3 |
| **Asset CDN** | Cloudflare R2 / AWS S3 + CloudFront | When assets > 50MB |

---

*This document should be updated as technology decisions evolve during implementation.*