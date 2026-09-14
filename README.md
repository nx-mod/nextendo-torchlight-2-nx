# torchlight-2

NEX game server for **Torchlight II** (Nintendo Switch, `010090400D366000`), built on the NextendoNetwork [nextendo-nex](https://github.com/NextendoNetwork/nextendo-nex) core. Source only: no binaries, no certs, no game assets. Not affiliated with Runic Games, Panic Button, Perfect World or Nintendo.

Work in progress: an untested scaffold. The server refuses to start until the game's NEX access key is known. See [NOTES.md](NOTES.md).

## Build

Clone this repo and `nextendo-nex` side by side, then:

    go build -o server.exe .

See `example.env` for configuration.

## Credits

- **[Nextendo Network](https://nextendo.network)**: the NEX core, gates, dashboard and server pattern this server follows (template: borderlands-1 / luigis-mansion-3).
- **[exlaunch](https://github.com/shadowninja108/exlaunch)** by **Shadow**: the in-game instrumentation used to map the game's online calls (`tl2-hack`).
- **[kinnay/NintendoClients](https://github.com/kinnay/NintendoClients)** and its [wiki](https://github.com/kinnay/NintendoClients/wiki): NEX protocol method ids and parameters, Switch error modules.
- **[Pretendo Network](https://pretendo.network)**: NEX documentation ([developer docs](https://developer.pretendo.network/overview/nex)).

References were read and reimplemented; no code was copied.