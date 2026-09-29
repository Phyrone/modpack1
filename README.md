# A Modpack

packwiz modpack for **Minecraft 1.21.1 / NeoForge 21.1.252**.

## Install

### PrismLauncher / MultiMC

1. Download [`packwiz-installer-bootstrap.jar`](https://github.com/packwiz/packwiz-installer-bootstrap/releases/latest/download/packwiz-installer-bootstrap.jar) into the instance's `minecraft/` folder.
2. Add a pre-launch command in **Edit Instance → Settings → Custom Commands** (PrismLauncher) or `instance.cfg`:

   ```text
   "$INST_JAVA" -jar packwiz-installer-bootstrap.jar -g https://raw.githubusercontent.com/Phyrone/modpack1/main/pack.toml
   ```

   Prefer the UI — it handles quoting. If editing `instance.cfg` directly, the quotes
   **must be escaped** (`\"`), otherwise Qt's `QSettings` strips them and the Java path
   (which contains spaces) is split, so launching fails with `execve: No such file or directory`:

   ```ini
   OverrideCommands=true
   PreLaunchCommand=\"$INST_JAVA\" -jar packwiz-installer-bootstrap.jar -g https://raw.githubusercontent.com/Phyrone/modpack1/main/pack.toml
   ```

### Server

```sh
java -jar packwiz-installer-bootstrap.jar -g -s server https://raw.githubusercontent.com/Phyrone/modpack1/main/pack.toml
```

## Server (Docker)

`docker-compose.yml` runs [itzg/docker-minecraft-server](https://docker-minecraft-server.readthedocs.io/en/latest/mods-and-plugins/packwiz/)
with packwiz enabled and a local `./data` bind mount:

```yaml
services:
  minecraft:
    image: itzg/minecraft-server:java21
    container_name: a-modpack-server
    restart: unless-stopped
    tty: true
    stdin_open: true
    ports:
      - "25565:25565" # game
      - "25575:25575" # RCON
    environment:
      EULA: "TRUE"
      TYPE: "NEOFORGE"
      VERSION: "1.21.1"
      NEOFORGE_VERSION: "21.1.252"
      PACKWIZ_URL: "https://raw.githubusercontent.com/Phyrone/modpack1/main/pack.toml"
      MEMORY: "${MEMORY:-8G}"
      MOTD: "A Modpack (packwiz)"
      ONLINE_MODE: "TRUE"
      VIEW_DISTANCE: "10"
      ENABLE_RCON: "true"
      RCON_PASSWORD: "${RCON_PASSWORD:-changeme}"
    volumes:
      - ./data:/data
```

Start it (all world/player/mod data lives in `./data`):

```sh
docker compose up -d
docker compose logs -f minecraft   # watch packwiz install + server startup
```

The container installs/updates the pack via `PACKWIZ_URL` on every start, honoring the
pack's `side` flags (only `server`/`both` mods). Override memory or the RCON password via a
local `.env` file or shell env (`MEMORY=12G RCON_PASSWORD=... docker compose up -d`).


## Editing

```sh
packwiz refresh      # regenerate index.toml after changing metadata/config
packwiz update --all # bump all mods
git add -A && git commit && git push
```

The `packwiz refresh` GitHub Action keeps `index.toml` up to date on every push.
