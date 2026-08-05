# ✈️ AeroArena - 3D Flying Combat Game

A web-based interactive 3D flying game where players pilot various flying vehicles through an obstacle-filled arena, destroying targets with equipped weapons.

## 🎮 Game Overview

**AeroArena** is a browser-based 3D arcade flying game featuring:
- **Vehicle Selection** - Choose from multiple unique flying vehicles (fighter jets, helicopters, flying soccer balls, and more)
- **3D Arena Combat** - Navigate a dynamic 3D environment filled with obstacles and targets
- **Weapon Systems** - Each vehicle has unique weapons (machine guns, missiles, curve shots, etc.)
- **Special Abilities** - Afterburners, hover mode, knuckleballs, and other unique mechanics
- **Progression System** - Unlock new vehicles by achieving high scores
- **Local Leaderboards** - Track your best runs and compete with yourself

## ✨ Key Features

### 🛩️ Vehicle Roster
| Vehicle | Category | Specialty |
|---------|----------|-----------|
| **F-16 Falcon** | Fighter Jet | High speed, afterburner boost, machine gun |
| **AH-64 Apache** | Attack Helicopter | High durability, homing missiles, hover capability |
| **Flying Soccer Ball ⚽** | Special | Extreme agility, curve shots, unpredictable knuckleball |
| **MQ-9 Reaper** | Drone | Stealth, laser weapon, silent running |
| **Jetpack Ranger** | Personal | Vertical flight, dash ability, dual pistols |

### 🎯 Gameplay Mechanics
- **Arcade Physics** - Accessible but skill-based flight model with momentum, drag, and lift
- **Destructible Obstacles** - Target-marked obstacles award points when destroyed
- **Combo System** - Rapid destructions multiply your score
- **Weapon Variety** - Hitscan, projectile, tracking, and curve-shot weapons
- **Special Abilities** - Cooldown-based unique mechanics per vehicle

### 🏆 Progression & Scoring
- **Score Components**: Obstacles destroyed × combo + survival time + distance traveled
- **Vehicle Unlocks** - Earn new vehicles by reaching score thresholds
- **Local Persistence** - All progress saved to SQLite database
- **Session History** - Review past runs with detailed stats

## 🛠️ Tech Stack Summary

| Layer | Technology |
|-------|------------|
| **Frontend** | React 18 + TypeScript + Vite |
| **3D Rendering** | Three.js + @react-three/fiber + @react-three/drei |
| **State Management** | Zustand |
| **Styling** | Tailwind CSS |
| **Backend** | FastAPI (Python) |
| **Database** | SQLite + SQLAlchemy 2.0 (async) |
| **Migrations** | Alembic |
| **3D Assets** | GLB/GLTF with DRACO compression |

## 📁 Documentation

- **[Tech Stack Details](docs/techstack.md)** - Complete technical architecture
- **[API Documentation](docs/API.md)** - REST endpoints and WebSocket events
- **[Asset Pipeline](docs/ASSET_PIPELINE.md)** - 3D model preparation workflow
- **[Deployment Guide](docs/DEPLOYMENT.md)** - Production deployment instructions

## 🚀 Quick Start

### Prerequisites
- Node.js 18+
- Python 3.11+
- Git

### Backend Setup
```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
alembic upgrade head
python -m app.seed
uvicorn app.main:app --reload --port 8000
```

### Frontend Setup
```bash
cd frontend
npm install
npm run dev
```

Open http://localhost:5173 to play!

## 🗺️ Development Roadmap

### Phase 1: Foundation ✅ (Planning)
- [x] Tech stack selection and architecture design
- [x] Database schema design
- [x] API contract definition
- [ ] Project scaffolding (backend + frontend)

### Phase 2: Core Gameplay
- [ ] Vehicle selection screen with 3D previews
- [ ] Arena environment (skybox, ground, lighting)
- [ ] Player controller with arcade physics
- [ ] Obstacle spawning and collision system
- [ ] Weapon/projectile systems
- [ ] HUD and game loop

### Phase 3: Polish & Content
- [ ] Vehicle models and animations
- [ ] Particle effects and audio
- [ ] Visual polish (bloom, screen shake, etc.)
- [ ] Settings and persistence
- [ ] Performance optimization

### Phase 4: Launch Ready
- [ ] End-to-end testing
- [ ] Production builds
- [ ] Deployment configuration
- [ ] Documentation completion

### Phase 5: Future Expansions
- [ ] Multiplayer (WebSocket sync)
- [ ] Additional vehicles and arenas
- [ ] Game modes (race, survival, co-op)
- [ ] Mobile touch controls
- [ ] Global leaderboards

## 📄 License

MIT License - Feel free to use for learning or personal projects!

---

**Built with ❤️ for web-based 3D gaming enthusiasts**