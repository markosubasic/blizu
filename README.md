# Blizu

Koliko je minuta hoda od tvoje adrese do liječnika, ljekarne, vrtića, škole, knjižnice, pošte, javnog prijevoza, parka i trgovine, za četiri persone (opće, starija osoba, roditelj s djetetom, invalidska kolica), i gdje bi nova ustanova koristila najviše ljudi. Grad Zagreb.

Rješenje za **Datathon Portala otvorenih podataka 2026.**, izazov A „Društvo i javne usluge”.
Izvorni kôd je pod licencom [EUPL-1.2](LICENSE).

| | |
| --- | --- |
| Video prezentacija | _poveznica se dodaje prije predaje_ |
| Radni prototip | _poveznica se dodaje nakon postavljanja na poslužitelj_ |
| Tehnička i projektna dokumentacija | [apps/blizu/docs/README.md](apps/blizu/docs/README.md) |
| Validacija | [apps/blizu/docs/validation.md](apps/blizu/docs/validation.md) |
| Odluke o arhitekturi i podacima | [docs/adr/](docs/adr/) |
| Izvori podataka s dokazima | [apps/blizu/data/evidence/](apps/blizu/data/evidence/) |

## Dokumentacija

Cijela dokumentacija je u [apps/blizu/docs/README.md](apps/blizu/docs/README.md). Sadrži:

- opis problema i korisnika,
- popis svih korištenih skupova podataka s izdavačem, licencom i poveznicom,
- tehničku arhitekturu (izvori → unos → PostgreSQL/PostGIS → FastAPI → React),
- opis API-ja (`/api/v1`, OpenAPI na `/openapi.json`),
- pokretanje, testove i poznata ograničenja.

Popis izvora s datumom dohvaćanja vidljiv je i u samoj aplikaciji na stranici „Izvori podataka”
(`GET /api/v1/sources`).

## Struktura repozitorija

```
apps/blizu/api    FastAPI + Typer CLI (blizu_api), Alembic migracije, testovi sa stvarnim snimljenim odgovorima
apps/blizu/web    React 18 + Vite + TypeScript + Tailwind (SPA)
apps/blizu/data   evidence/ (sirovi odgovori izvora s datumom)
apps/blizu/docs   dokumentacija aplikacije
packages/core     opendata_core: konfiguracija, HTTP s cacheom, provenijencija izvora, CKAN, geo, geokodiranje,
                  mreža stanovništva, optimizacija, e-pošta, tvornica FastAPI aplikacije
packages/ui       @dt/ui: tokeni, Tailwind preset, pristupačne React komponente
infra/            slika baze (PostGIS + pg_trgm + pgvector), Dockerfile za API i SPA
docs/             FOUNDATION.md (konvencije), adr/ (odluke)
```

## Brzo pokretanje

Potrebno: Docker, [uv](https://docs.astral.sh/uv/) (Python 3.12), Node 20+ i pnpm 9, make.
Na Windowsu `make` pokrenite iz Git Basha i koristite `127.0.0.1`, a ne `localhost`.

```bash
cp .env.example .env              # postavite POSTGRES_PASSWORD i CONTACT_EMAIL (DB_PORT ako je 5432 zauzet)
make dev                          # PostgreSQL 16 + PostGIS, Mailpit (http://127.0.0.1:8025)
uv sync && pnpm install
make migrate
make ingest STEP=<korak>          # koraci i redoslijed: apps/blizu/docs/README.md
uv run --package blizu-api uvicorn blizu_api.main:app --port 8010
BLIZU_API_URL=http://127.0.0.1:8010 pnpm --filter blizu-web dev   # http://127.0.0.1:5173
```

Testovi: `make test` (pytest bez mrežnih poziva + vitest). Produkcija: `docker-compose.prod.yml` + Caddy, vidi
dokumentaciju aplikacije.

## Odluke

- [0001-monorepo-and-stack.md](docs/adr/0001-monorepo-and-stack.md)
- [0002-pgdg-archive-for-pgvector.md](docs/adr/0002-pgdg-archive-for-pgvector.md)
- [0003-js-tooling-versions.md](docs/adr/0003-js-tooling-versions.md)
- [0004-per-ip-rate-limit-middleware.md](docs/adr/0004-per-ip-rate-limit-middleware.md)
- [0005-geofabrik-fallback-for-osm-features.md](docs/adr/0005-geofabrik-fallback-for-osm-features.md)
- [0009-blizu-night-atlas.md](docs/adr/0009-blizu-night-atlas.md)

## Korišteni alati

Rješenje koristi isključivo softver otvorenog koda (Python, FastAPI, SQLAlchemy, PostgreSQL/PostGIS, React, Vite,
MapLibre GL i ostalo navedeno u `pyproject.toml` i `package.json`). Za razvoj je korišten alat temeljen na umjetnoj
inteligenciji **Claude Code** (Anthropic); radna pravila za taj alat su u [CLAUDE.md](CLAUDE.md).

## Tim

Ivan Stipić, Marko Subašić. Mentor: _upisati_.

Ovaj repozitorij je izdvojen iz zajedničkog repozitorija
[istipic/datathon-2026](https://github.com/istipic/datathon-2026) s očuvanom poviješću commitova; `packages/core`,
`packages/ui` i `infra/` dijele se s ostalim aplikacijama tima.
