# AguaLibreOS Repository Guidelines

## Project Structure & Module Organization

AguaLibreOS is an Arch Linux-based lightweight OS for autonomous water-delivery drones in Mexicali, built from a live ISO (`airootfs/`, `build-iso/`) with a custom Rust `#![no_std]` kernel (`kernel/`) and a React ground-station dashboard (`frontend/`).

The kernel is split into three crates: `drone_control` (PID/flight), `navigation` (SLAM, GPS-RTK, CV), and `safety` (fail-safes, RTH).  
`modules/` holds flight payload, energy, comms, AI routing (QAOA-inspired), and climate adaptation.  
`api/` (FastAPI) provides MAVLink telemetry; `frontend/` provides the fleet dashboard.  
`config/` holds YAML/JSON system config; `docs/` has ISO build and AFAC-compliance docs; `simulations/` wraps Gazebo/AirSim/QEMU.

## Build, Test, and Development Commands

- Full build: `make all`
- Kernel only: `make build-kernel` (runs `cargo build --release` in `kernel/`)
- API only: `cd api && pip install -r requirements.txt`
- Frontend only: `make build-frontend` (npm install + build in `frontend/`)
- ISO build: `make build-iso` (Archiso via `mkarchiso`)
- Run simulator: `make sim` (Docker Compose in `simulations/`)
- Full test suite: `make test` (cargo + pytest + vitest)
- Single test examples:
  - Kernel: `cd kernel && cargo test drone_control`
  - API: `cd api && pytest tests/test_telemetry.py`
  - Frontend: `cd frontend && npx vitest run src/lib/mavlink.test.ts`

## Coding Style & Naming Conventions

- **Rust (kernel):** rustfmt default; `UPPER_SNAKE_CASE` for constants, `snake_case` for functions/types, `CamelCase` for modules.
- **Python (api):** Black formatter, isort; `snake_case` for files, functions, variables; `PascalCase` for classes. Use Pydantic v2 models in `api/models/`.
- **TypeScript (frontend):** strict mode (`tsconfig.json`); `PascalCase` for React components, `camelCase` for hooks/utils, `UPPER_SNAKE_CASE` for constants. Use ESLint with `@typescript-eslint`.
- **YAML/JSON (config):** lowercase keys, hyphen-separated, no tabs.

## Testing Guidelines

- **Kernel tests** go in `kernel/*/tests/` or inline `#[cfg(test)]` modules.
- **API tests** in `api/tests/` with pytest + HTTPX async client.
- **Frontend tests** in `frontend/src/**/__tests__/*.test.ts` with Vitest + Testing Library.
- **Sim tests** in `simulations/`; prefer headless runs (`GAZEBO_HEADLESS=1`).
- Target ≥ 80% coverage for safety-critical `safety/` and `drone_control/` crates.

## Commit & Pull Request Guidelines

- No commits exist yet; follow conventional commits: `feat(module): short description`, `fix(drone_control): ...`, `docs: ...`.
- PRs should include: summary of changes, affected modules, simulator test results, and AFAC compliance notes for flight-related changes.

## Safety & Flight Rules

- Never modify `safety/` or `payload/` logic without simulator validation (`make sim`) and an accompanying `tests/` update.
- Geo-fencing coordinates for authorized Mexicali colonias belong in `config/geofence.yaml`; changes here require code review.
- All MAVLink message handling in `modules/comms/` must handle packet loss gracefully (timeout-based fallbacks).
