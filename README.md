# BigLobby3 (Diesel 3.0 port)

BigLobby: No PDMod Required!

This is a fork of [Crackdown-PD2/BigLobby3](https://github.com/Crackdown-PD2/BigLobby3) updated for the
PAYDAY 2 Diesel 3.0 / 64-bit engine (open beta, June 2026, shipping to the default branch on 2026-08-13).
Upstream 3.27.6 targets the Diesel 2 engine and breaks on Diesel 3.0.

## Requirements

- PAYDAY 2 on the Diesel 3.0 engine (`open_beta` branch during the beta, default branch afterwards)
- A 64-bit SuperBLT DLL (`WSOCK32.dll`) plus the SuperBLT Lua basemod in `mods/base`:
  [diesel-modding/PAYDAY2-SuperBLT](https://github.com/diesel-modding/PAYDAY2-SuperBLT) and
  [diesel-modding/PAYDAY2-SuperBLT-Lua](https://github.com/diesel-modding/PAYDAY2-SuperBLT-Lua)
- Every player in the lobby needs the same SuperBLT build and the same BigLobby version. Mixed versions
  cannot connect, and neither can a Diesel 2 client and a Diesel 3 client.

Set the lobby size in Mod Options (BigLobby), it defaults to 22 and is capped at 128 by the injected
network messages. Achievements are disabled automatically above 4 players.

## What the Diesel 3.0 port changes

Engine-side renames and API changes that broke the mod's copied vanilla code:

- `SystemInfo:platform() == Idstring("WIN32")` and friends replaced by the engine globals
  `IS_WIN32` / `IS_PC` / `IS_XB1` / `IS_PS4`, and `SystemInfo:distribution()` /
  `SystemInfo:matchmaking()` by `IS_STEAM` / `IS_EPIC` / `IS_STEAM_MM`. Those globals come from
  `Distribution:type()` and `DistributionMatchmaking:type()`, the platform-agnostic interfaces that
  replaced the separate Steam and EpicMM ones.
- `SocialHubFriends:is_blocked()` is now `managers.socialhub:is_user_blocked()`, and
  `SocialHubFriends:is_friend_global()` is now
  `managers.socialhub:is_user_socialhub_or_distribution_friend()`.
- `NetworkPeer:create_ticket(account_id)` is asynchronous: it takes a callback as a second argument
  and the auth ticket may have to be sent in chunks through `TDVS`. Both `HostStateInLobby` and
  `HostStateInGame` join paths were rewritten accordingly, otherwise no client can ever finish joining.
- `tweak_data.weapon[id].category` is gone, lobby poses now read `categories[1]`.

Fixed along the way, unrelated to the engine change:

- `HostStateInGame:on_join_request_received` denied blocked users with
  `JOIN_REPLY.SHUB_BLOCKE` (typo, evaluates to `nil`), now `SHUB_BLOCKED`.
- `HostStateInLobby:on_join_auth_received` passed a stale extra argument to
  `_introduce_new_peer_to_old_peers`, which shifted `xuid` and `xnaddr` by one slot.

Verified unchanged against the Diesel 3.0 scripts: the other 42 overridden functions, all 39 hook
targets, the whole `biglobby__*` RPC table (no handler signature changed between Diesel 2 and Diesel 3),
and the matchmaking hook (`NetworkMatchMakingSTEAM` is still what PC instantiates, `OPEN_SLOTS` and
`_BUILD_SEARCH_INTEREST_KEY` still exist).

# Credits:
- Polarathene: Original version of the mod located here: https://github.com/polarathene/biglobby
- steam-test1: Contributor to current/previous project
- ZNix: SuperBLT creator, XML injection API used to obsolete the pdmod file
- RESTORATION Mod team: Additional R&D
