# torchlight-2

NEX game server for **Torchlight II** (Nintendo Switch, `010090400D366000`), built on the NextendoNetwork [nextendo-nex](https://github.com/NextendoNetwork/nextendo-nex) core. Source only: no binaries, no certs, no game assets. Not affiliated with Runic Games, Panic Button, Perfect World or Nintendo.

## Status

Tested on **one console only** (a CFW Switch, 2026-09-13). There has been no two-sided test: no second player has ever joined.

- **Seen working live:** the game's server id (`0x2e608000`) and NEX access key (`ebf6d32e`, the default here), the auth and secure logins, and creating a session (`CreateMatchmakeSessionWithParam`).
- **Known problem, not re-tested:** in that session the player list did not show the host correctly. The reply to `FindMatchmakeSessionByParticipant` was implemented afterwards, but it has not been run against the game again.
- **Not tested:** joining, leaving, and any second player. The game also has Vivox voice, which needs nothing from this server.

See [NOTES.md](NOTES.md) for the calls observed and how the key and server id were found.

## Build

Clone this repo and `nextendo-nex` side by side, then:

    go build -o server.exe .

See `example.env` for configuration.

## Credits

- **[Nextendo Network](https://nextendo.network)**: the NEX core, gates, dashboard and server pattern this server follows (template: borderlands-1 / luigis-mansion-3).
- **[exlaunch](https://github.com/shadowninja108/exlaunch)** by **Shadow**: the in-game instrumentation used to map the game's online calls (`tl2-hack`, an exlaunch logging module that is not part of this repository).
- **[kinnay/NintendoClients](https://github.com/kinnay/NintendoClients)** and its [wiki](https://github.com/kinnay/NintendoClients/wiki): NEX protocol method ids and parameters, Switch error modules.
- **[Pretendo Network](https://pretendo.network)**: NEX documentation ([developer docs](https://developer.pretendo.network/overview/nex)).

References were read and reimplemented; no code was copied.