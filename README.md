# torchlight-2

NEX game server for **Torchlight II** (Nintendo Switch, `010090400D366000`), built on the NextendoNetwork [nextendo-nex](https://github.com/NextendoNetwork/nextendo-nex) core. Source only: no binaries, no certs, no game assets. Not affiliated with Runic Games, Panic Button, Perfect World or Nintendo.

## Status

Tested on **one console only** (a CFW Switch, 2026-09-13). There has been no two-sided test: no second player has ever joined.

- **Seen working live:** the game's server id (`0x2e608000`) and NEX access key (`ebf6d32e`, the default here), the auth and secure logins, and creating a session (`CreateMatchmakeSessionWithParam`).
- **Known problem, not re-tested:** in that session the player list did not show the host correctly. The reply to `FindMatchmakeSessionByParticipant` was implemented afterwards, but it has not been run against the game again.
- **Not tested:** joining, leaving, and any second player. The game also has Vivox voice, which needs nothing from this server.

See [NOTES.md](NOTES.md) for the calls observed and how the key and server id were found.

## Requirements

This server runs behind the rest of the Nextendo stack. It needs nothing cloned next to it: `go build` fetches the NEX core ([nextendo-nex](https://github.com/NextendoNetwork/nextendo-nex), a Go module) by itself.

| component | needed? | what it must provide |
|---|---|---|
| **sni-router** | required | A route sending `g2e608000-lp1.s.n.srv.nintendo.net` to `BACKEND_TL2` (default `127.0.0.1:8458`). The route is not in sni-router's `main` yet: it is the `feat/torchlight-2` branch of [nx-mod/sni-router](https://github.com/nx-mod/sni-router), one commit on top of `main`. |
| **nextendo-account** | required with `NEXTENDO_REQUIRE_ACCOUNT=1` | `GET /api/nsa` (a console's NSA id to a Nextendo account) and `POST /internal/online-check`, with `X-Internal-Key`. With the gate on, a login whose NSA id cannot be resolved is refused. |
| **nextendo-dashboard** | optional | A `tl2` source polling `/api/stats` on port 8096 (`DASH_TL2_URL`, `DASH_TL2_TOKEN`): the `feat/torchlight-2-stats` branch of [nx-mod/nextendo-dashboard](https://github.com/nx-mod/nextendo-dashboard). Without it the server works but is not on the shared dashboard. |
| **DNS** | required | `g2e608000-lp1.s.n.srv.nintendo.net` must resolve to the machine running sni-router, and never to Nintendo. On a console that is an Atmosphere hosts entry; the standard Nextendo hosts file already sends `*.srv.nintendo.net` to the stack. |
| **TLS certificate** | required | A certificate and key for the game's auth host, from a CA your clients trust (`CERT_FILE`, `KEY_FILE`). Yours to provide; none is shipped. |

The secure server is not behind the router: the game connects to `NEXTENDO_HOST:60014` directly, so `NEXTENDO_HOST` must be the address players can reach.

## Install

1. Build: `go build -o server.exe .` (Go 1.23 or later).
2. Copy `example.env` to `.env` and set the secrets. The server does **not** read `.env` itself: export the variables into its environment (a launcher or service file), and set `NEXTENDO_SECRET` or `NEXTENDO_SECRET_FILE`, `NEXTENDO_INTERNAL_KEY`, `NEXTENDO_SECURE_PASSWORD` and `DASH_TOKEN`.
3. Put `cert.pem` and `key.pem` next to the binary, or point `CERT_FILE` and `KEY_FILE` at them.
4. Start it. Auth listens on `AUTH_PORT` (`8458` behind sni-router), the secure server on `60014`, the dashboard on `8096`.

| setting | default | meaning |
|---|---|---|
| `TL2_ACCESS_KEY` | `ebf6d32e` | the game's NEX access key, confirmed live |
| `TL2_NEX_VERSION` | `40000` | the value in use; it has not been confirmed against the game |

Game: Torchlight II, title `010090400D366000`, game server id `0x2e608000`.

## Known limits

- **One console only.** No second player has ever joined.
- **Player list.** In the one session run, the game's player list did not show the host correctly. The reply to `FindMatchmakeSessionByParticipant` was implemented afterwards; it has not been run against the game since.
- **Unhandled calls** are logged as `[TL2 Secure] UNHANDLED ...` with the full request and answered with an empty success, so the game may carry on as if the call had worked. The secure server registers the core's matchmaking, matchmaking-extension, NAT traversal, ranking and utility handlers.
- **NEX version** (`TL2_NEX_VERSION`) is 4.0.0 as a working value; the game's real build has not been confirmed.
- **Pia.** The game also uses Pia for peer-to-peer play; this server does not touch it. Voice chat (Vivox) needs nothing from this server.
- No persistence: games and connections live in memory.

## To do

- Run a second player through browse, join and leave, and check the player list after the `FindMatchmakeSessionByParticipant` change.
- Confirm the NEX version the game was built with.
- Add tests (there are none yet).

## Credits

- **[Nextendo Network](https://nextendo.network)**: the NEX core, gates, dashboard and server pattern this server follows (template: borderlands-1 / luigis-mansion-3).
- **[exlaunch](https://github.com/shadowninja108/exlaunch)** by **Shadow**: the in-game instrumentation used to map the game's online calls (`tl2-hack`, an exlaunch logging module that is not part of this repository).
- **[kinnay/NintendoClients](https://github.com/kinnay/NintendoClients)** and its [wiki](https://github.com/kinnay/NintendoClients/wiki): NEX protocol method ids and parameters, Switch error modules.
- **[Pretendo Network](https://pretendo.network)**: NEX documentation ([developer docs](https://developer.pretendo.network/overview/nex)).

References were read and reimplemented; no code was copied.

## Credits

Built by nx-mod for the **Nextendo Network**, on the work of the Nextendo Network team — https://nextendo.network. Nextendo is awesome.
