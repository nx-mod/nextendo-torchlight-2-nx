# Torchlight II (Switch) - Nextendo server: notes

Keep this file updated as you go: it is the map for this server.

- Game: Torchlight II, title `010090400D366000` (**to confirm** from the binary: strings / NPDM).
- Binary: dumped to `sd:/atmosphere/contents/010090400D366000/exefs/main` (2026-09-13). An exefs
  override loads at boot, so it must stay the game's own, unmodified main. Copy + segments go
  in `tl2-hack/capture/nso/` (git-ignored): `go run ./tools/nsoextract <main> capture/nso`.
- Status (2026-09-13): **scaffold only, never run against the game.** Assumed NEX (like
  Borderlands) until the binary or tl2-hack says otherwise.

## To find

1. **Online backend.** Look for `nn::nex` symbols / `SetSandboxAccessKey` in the strings.
   No NEX means a custom backend (HTTPS or its own sockets, as Diablo III's Demonware): then
   this NEX scaffold is the wrong base and the server follows `diablo-3` instead.
2. **Access key**: tl2-hack logs `SetSandboxAccessKey`. Set `TL2_ACCESS_KEY`.
3. **Game server id**: tl2-hack logs `nsd resolve 'g<id>-%.s.n.srv.nintendo.net'`. Add the
   host to sni-router (`BACKEND_TL2=127.0.0.1:8458`) and to the Switch hosts files.
4. **NEX version**, then the RMC calls the game makes: unhandled ones are logged in full
   (`[TL2 Secure] UNHANDLED ...`).

## Ports (local stack)

| What | Port |
|---|---|
| auth (behind sni-router) | 8458 |
| secure | 60014 |
| dashboard | 8096 |

## Logger

`tl2-hack` (this folder, its own git repository, ignored here) is an exlaunch module:
build with devkitPro (`make`), copy `deploy/subsdk9` + `deploy/main.npdm` to
`sd:/atmosphere/contents/010090400D366000/exefs/`, play online, close the game, pull
`sd:/config/tl2-hack/log.txt`.

## References

- kinnay/NintendoClients wiki (NEX protocols), Pretendo developer docs, exlaunch.
- Sibling servers: `borderlands-1` (NEX, same pattern), `diablo-3` (custom Demonware backend).