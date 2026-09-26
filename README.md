# Eliocraft Server

Binary-only releases of the Eliocraft dedicated server. Source remains
private; this repository holds only compiled server binaries and checksums.

Eliocraft is a voxel survival game with territory claims, factions,
sieges, procedural worlds, wildlife, minigames, and an authoritative plugin
system. A network gateway (port `25599`) routes players to hub, survival, and
minigame backends. The dedicated server is the authoritative simulation for the
game client `Eliocraft`.

## System requirements

- **Windows x64** or **Linux x64**. Releases are self-contained, so no .NET
  runtime or other prerequisite is required.
- A stable TCP port (default `25599`).

## Releases

Each release publishes:

- `ChunkSurvival-Server-<version>-linux-x64.tar.gz`
- `ChunkSurvival-Server-<version>-win-x64.zip`
- `SHA256SUMS.txt`

Download the latest release from the
[Releases](https://github.com/stoxello/ChunkSurvival-Server/releases) page.
Verify the archive against `SHA256SUMS.txt` before running it.

## Quick start

### Windows

```powershell
Expand-Archive ChunkSurvival-Server-<version>-win-x64.zip -DestinationPath server
cd server
.\ChunkSurvival.Server.exe 25599
```

### Linux

```bash
mkdir -p server && tar -xzf ChunkSurvival-Server-<version>-linux-x64.tar.gz -C server
cd server
chmod +x ChunkSurvival.Server
./ChunkSurvival.Server 25599
```

The server writes a default `server.cfg` on first launch. Type `quit` to stop;
survival worlds are saved on a graceful stop.

## Configuration

By default the server reads `server.cfg` in its working directory. Set the
`SERVER_CONFIG` environment variable to point at a different file (used by
Docker to keep config in a bind-mounted `/cfg`). A default config is written
if the file does not exist.

Key settings:

| Setting | Meaning |
| --- | --- |
| `role=survival` | `survival`, `hub`, or `minigame`. Survival persists its world; hub/minigame load an authored `map` and reset each restart. |
| `seed=1337` | Procedural world seed (survival). |
| `map=` | Prebuilt map file to load for hub/minigame. Blank generates from the seed. |
| `mobs=on` | Spawn mobs (default on for survival). |
| `chunkclaims=on` | Territory system. `off` = free-roam sandbox without claims, sieges, or expansion. |
| `factions=on` | Faction claims, alliances, and shared cartography. |
| `weather=on` / `timechange=on` | Weather and day/night cycle toggles. |
| `time=` | Fixed starting time of day (`noon`, `18:00`, or a `0..1` fraction). |
| `spawn=x,y,z` | Fixed spawn for hub/minigame. |
| `portal=<server>:<x1,y1,z1>:<x2,y2,z2>` | Repeatable transfer boxes to named backends via the gateway. |
| `chunkFormDelaySeconds`, `siegeDays`, `dayLengthSeconds` | Territory pacing. |
| `maxPlayers`, `minPlayers`, `roundCountdownSeconds`, ... | Population cap and minigame timing. |
| `serverId=`, `serverKey=` | Credentials for mandatory server authorization (below). |

## Server authorization

Player-hosted servers must authorize against the Eliocraft account service.
Authorization is mandatory: set `serverId` and `serverKey` in `server.cfg`, or
supply `SERVER_ID` and `SERVER_KEY` (plus `AUTHORIZATION_URL`, default
`https://id.chunksurvival.com`) as environment variables. Register at
[id.chunksurvival.com/servers](https://id.chunksurvival.com/servers) to obtain
credentials. Credentials are never written to logs.

## Plugins

Official and community plugins are .NET 9 assemblies loaded from `Plugins/`
(overridable with `PLUGINS_PATH`; writable plugin data defaults there too, or
to `PLUGIN_DATA_PATH`). Each plugin lives in its own directory with a
`plugin.yml` manifest:

```text
Plugins/
└── Spleef/
    ├── plugin.yml
    ├── BlockGame.Plugins.Spleef.dll
    └── BlockGame.PluginApi.Minigames.dll
```

Do not ship `BlockGame.PluginApi.dll`; the server supplies it. Restart the
server after installing or updating plugins. Plugins run in-process, so only
install plugins you trust.

A minigame backend is configured with `role=minigame` plus `minigame=<id>`
(e.g. `spleef`). Official plugins are distributed through the
[Eliocraft Plugins](https://github.com/stoxello/ChunkSurvivalPlugins)
repository.

## Running a multi-server cluster (Docker)

The gateway + backend cluster is published as container images on GitHub
Container Registry (`blockgame-gateway` and `blockgame-server`). The gateway is
the only public game port; backends register by internal hostname and role
config. Set `REQUIRE_AUTHORIZATION=true`, `AUTHORIZATION_URL`, `SERVER_ID`, and
`SERVER_KEY` from your registered server credentials, then start the stack
(see the repository's `docker-compose-server.yml` for the full topology).

## Building from source

Source is not published. Releases are built from the private source repository
and tagged `server-vX.Y.Z`; the tag version must match the server project
version. The workflow runs server integration tests, publishes self-contained
Linux x64 and Windows x64 builds, and attaches both archives plus
`SHA256SUMS.txt` here.
