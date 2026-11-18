# Visit counter

A page that shows how many times it was visited.

```
browser ──► site (nginx, the page in site/) 
        └─► api  (Flask, api/app.py, GET /visits) ──► db (Redis, keeps the count in a volume)
```

## On my laptop

```bash
docker compose up --build
```

Open http://localhost:8081 (the API answers on http://localhost:8082/visits).

## Online

It runs on my practice server at https://student02.wt.thomasjuldo.com/visits/, next to [music-map](https://github.com/oscarcaux7/music-map) (at `/`). One Caddy, in its own repo [server-proxy](https://github.com/oscarcaux7/server-proxy), serves both: `/visits/api/*` goes to the API, the rest of `/visits/*` to the page.

- `compose.server.yaml` is added on top of `compose.yaml` on the server only: nothing publishes a port, the page and the API join the Docker network `web` (as `visit-counter-site` and `visit-counter-api`), Redis stays on this project's own network, and everything restarts by itself (`restart: unless-stopped`).
- The page calls the API with a relative address (`api/visits`), so it works under `/visits/`.
- There is no secret, so no `.env`.

Start it on the server (the first time; `server-proxy`'s README covers the whole server):

```bash
docker network create web          # once per server, if it does not exist yet
git clone https://github.com/oscarcaux7/visit-counter.git
cd visit-counter
docker compose -f compose.yaml -f compose.server.yaml up -d --build
```

**Deploy a change:** commit and push from the laptop, then on the server:

```bash
cd ~/visit-counter && git pull && docker compose -f compose.yaml -f compose.server.yaml up -d --build
```

This never restarts Caddy or music-map. The count is kept in the volume `counter-data`. If the server dies the count is lost (unless the volume was backed up), and the counter starts again from 0.
