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
    stop_grace_period: 2m
    ports:
      - "25565:25565" # game only; RCON (25575) stays internal
    environment:
      EULA: "TRUE"
      TYPE: "NEOFORGE"
      VERSION: "1.21.1"
      NEOFORGE_VERSION: "21.1.252"
      PACKWIZ_URL: "https://raw.githubusercontent.com/Phyrone/modpack1/main/pack.toml"
      ENABLE_RCON: "true" # internal only (not published); needed for the direct console
      JVM_OPTS: >-
        -XX:+UnlockExperimentalVMOptions
        -XX:+UseZGC
        -XX:+ZGenerational
        -XX:+AlwaysPreTouch
        -XX:+DisableExplicitGC
        -XX:+PerfDisableSharedMem
        -XX:ZUncommitDelay=300
      USE_AIKAR_FLAGS: "false"
      MEMORY: "${MEMORY:-8G}"
      MOTD: "A Modpack (packwiz)"
      ONLINE_MODE: "TRUE"
      VIEW_DISTANCE: "10"
      ALLOW_FLIGHT: "TRUE" # disable vanilla "flying is not enabled" kicks
    volumes:
      - ./data:/data
```

Start it (all world/player/mod data lives in `./data`):

```sh
docker compose up -d
docker compose logs -f minecraft   # watch packwiz install + server startup
```

The container installs/updates the pack via `PACKWIZ_URL` on every start, honoring the
pack's `side` flags (only `server`/`both` mods). Override memory via a local `.env` file or
shell env (`MEMORY=12G docker compose up -d`).

### Console

RCON stays enabled (it drives mc-server-runner's direct stdin wiring) but its port is **not
published**, so it isn't reachable from outside. In this mode mc-server-runner hands the
container TTY straight to the JVM, so attaching gives the *native* server console with
colors and tab-completion:

```sh
docker compose attach minecraft   # detach with Ctrl-p Ctrl-q
```

Do **not** set `CREATE_CONSOLE_IN_PIPE`, or the TTY is replaced by a pipe and the native
console (colors/completions) is lost. No itzg event hooks (`RCON_CMDS_*`) are configured.

`JVM_OPTS` uses generational ZGC; it requires Java 21 (the `java21` image) and pairs with
`MEMORY`.


## Editing

```sh
packwiz refresh      # regenerate index.toml after changing metadata/config
packwiz update --all # bump all mods
git add -A && git commit && git push
```

The `packwiz refresh` GitHub Action keeps `index.toml` up to date on every push.
