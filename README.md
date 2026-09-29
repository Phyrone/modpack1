# A Modpack

packwiz modpack for **Minecraft 1.21.1 / NeoForge 21.1.252**.

## Install

### PrismLauncher / MultiMC

1. Download [`packwiz-installer-bootstrap.jar`](https://github.com/packwiz/packwiz-installer-bootstrap/releases/latest/download/packwiz-installer-bootstrap.jar) into the instance's `minecraft/` folder.
2. Add a pre-launch command to `instance.cfg`:

   ```ini
   OverrideCommands=true
   PreLaunchCommand="$INST_JAVA" -jar packwiz-installer-bootstrap.jar -g "https://raw.githubusercontent.com/Phyrone/modpack1/main/pack.toml"
   ```

### Server

```sh
java -jar packwiz-installer-bootstrap.jar -g -s server https://raw.githubusercontent.com/Phyrone/modpack1/main/pack.toml
```

## Editing

```sh
packwiz refresh      # regenerate index.toml after changing metadata/config
packwiz update --all # bump all mods
git add -A && git commit && git push
```

The `packwiz refresh` GitHub Action keeps `index.toml` up to date on every push.
