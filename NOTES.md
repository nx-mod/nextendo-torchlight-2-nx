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
## Binary findings (2026-09-13, main pulled over FTP)

- Title `010090400D366000` **confirmed Torchlight II** (build path `C:\Torch\Release\builds\sourcecode\Delvers\NX64\Shipping`, Runic Games).
  SHA-256 `06F685D885BE4A8D68EAF8E5EBAF127795C1889341D3759F5D1639422B4E4595`; text @0, rodata @0x01DFC000, data @0x02302000.
- **Online is NEX + Pia**, linked statically with names stripped: `NexMatchRandomMatchmakeJob`, `NexMatchBrowseMatchmakeJob`,
  `NexMatchJointSessionJob`, `RendezVousLogout`, "Pia Send", hosts `g%08x-%%.s.n.srv.nintendo.net`,
  `nncs1-/nncs2-%.n.n.srv.nintendo.net`, `%s.%%.p.srv.nintendo.net`. So this NEX scaffold is the right base.
  Also Vivox voice (`v4mt1s.www.vivox.com`), which needs nothing from Nextendo.
- No `SetSandboxAccessKey` symbol (stripped), so tl2-hack cannot hook it by name. **Access key candidates** (the only
  8-hex strings in rodata): `0a4f113b`, `ebf6d32e`. Try them in `TL2_ACCESS_KEY`; a wrong key fails the PRUDP handshake.
- SDK imports present: `nn::nsd::ResolveEx` (tl2-hack logs the game server id), sockets (both variants), `nn::fs::SetAllocator`.
  Not imported: nn::ssl, curl, bcat, socket Read/Write. Error popups use `nn::err::ShowError(ErrorResultVariant const&)`.
## First live run (2026-09-13, CFW Switch, tl2-hack)

- **Game server id `0x2e608000`**: `nsd resolve 'g2e608000-%.s.n.srv.nintendo.net'` -> `g2e608000-lp1.s.n.srv.nintendo.net`,
  getaddrinfo -> 192.168.137.1, connect :443 from main+0x1ca5e18, then `ssl SetHostName` (NEX over WebSocket/TLS).
  Retried three times. The u32 is not stored as a plain constant in text/rodata/data.
  sni-router had no route, so the connection fell to baas-proxy (BACKEND_DEFAULT) and the game showed online error popups.
  Now routed: `BACKEND_TL2=127.0.0.1:8458` (sni-router commit "Route Torchlight II ...").
- Also resolved `2e608000.%.p.srv.nintendo.net` (no result): the Pia/P2P host pattern.
- **Access key: `ebf6d32e`** (not yet confirmed live). Of the two 8-hex strings in rodata, `0a4f113b` sits inside the
  Vivox SIP strings (voice chat), while `ebf6d32e` (VA 0x1F85966) sits among the game's own strings next to
  `m_currentServer.IsValid()` and `core::LOBBY_ATTEMPT_CONNECT`. Set in the launcher as `TL2_ACCESS_KEY`.
- tl2-hack: 25 hooks installed; nn::nex symbols are stripped (not found). The first build crashed the game at launch:
  exlaunch's hook JIT pool only fit 20 trampolines (fixed in tl2-hack, JitSize 0x4000).