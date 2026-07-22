# VoidSoulSpigot

Fork de [PandaSpigot](https://github.com/hpfxd/PandaSpigot) (que a su vez es fork de Paper) para Minecraft **1.8.8**,
enfocado en **rendimiento de chunks** y **mecánicas de PvP** configurables, inspirado en las opciones de forks
competitivos como Azurite.

## Características propias (además de todo lo de PandaSpigot)

- **`voidsoul.yml`**: archivo de configuración propio, con secciones por mundo (igual que `pandaspigot.yml`).
- **Marca configurable** (`serverBrandName`): el nombre que aparece en el F3 del cliente y en `/version`.
- **Sistema de knockback extendido** (por mundo, sección `knockback`):
    - `friction`, `horizontal`, `vertical`, `verticalLimit`, `extraHorizontal`, `extraVertical`
    - `slowdown`: multiplicador de frenado del atacante al conectar un golpe con knockback (vanilla `0.6`)
    - Knockback de **caña de pescar** (`rodHorizontalMultiplier`, `rodVerticalMultiplier`)
    - ⚠️ El knockback se configura aquí y **reemplaza** la sección `knockback` de `pandaspigot.yml`.
- **Hit delay configurable** (`combat.hitDelayTicks`): ticks de invulnerabilidad entre golpes (vanilla `20`).
- **Fixes de bugs de combate** (activables):
    - `combat.fixEatWhileSprinting`: evita mantener el sprint mientras se consume comida/pociones/leche
      (no afecta al bloqueo con espada, el block-hit sigue funcionando).
    - `combat.fixDropEatDesync`: suelta el ítem en uso correctamente al dropearlo mientras se come.
- **Trayectoria de proyectiles configurable**: aleatoriedad de bolas de nieve, huevos, perlas y pociones
  (`projectiles.snowballRandomness`, `eggRandomness`, `pearlRandomness`, `potionRandomness`;
  `1.0` = vanilla, `0.0` = trayectoria exacta).
- **Daño de enderpearl configurable** (`projectiles.pearlDamage`, vanilla `5.0`, `0` lo desactiva).
- **Chunks**:
    - `chunks.maxUnloadsPerTick`: límite de chunks descargados por tick (vanilla/Spigot `100`).
    - `chunks.maxAutosaveChunksPerPass`: reactiva el límite incremental de autosave que Spigot
      eliminó (default `200`, `0` = comportamiento Spigot sin límite). El apagado y `/save-all`
      nunca se limitan.
    - Toggles de tick por mundo: `ticking.tickHoppers`, `tickFurnaces`, `tickBrewingStands`,
      `tickBeacons`, `tickVillages`.

Todo lo de PandaSpigot sigue disponible: Netty 4.1, tick loop moderno, knockback base, eventos backporteados
(`PlayerChunkLoadEvent`, `EntityMoveEvent`, etc.), Adventure API y más. Ver [patches](./patches/).

## Compilar

Requisitos: **JDK 17+** (probado con JDK 21), **Git** y **Bash** (en Windows, Git Bash).

```bash
./panda jar
```

El jar final queda en `VoidSoulSpigot-paperclip.jar`.

Comandos útiles del script de build (`./panda`):

| Comando | Descripción |
| --- | --- |
| `setup` | Remapea, decompila y aplica todos los parches. |
| `p`, `patch` | Aplica los parches sin compilar. |
| `j`, `jar` | Aplica parches y compila; genera el paperclip jar. |
| `rb`, `rebuild` | Regenera los archivos de parche desde los work trees. |
| `e server` / `e api` | Edita un parche existente (rebase interactivo). |

El flujo de trabajo de parches es el mismo de Paper/PandaSpigot: los cambios se hacen en los work trees
generados `PandaSpigot-Server` / `PandaSpigot-API`, se commitean ahí, y `./panda rb` los guarda como
archivos en `patches/`.

## API

La API es compatible con PandaSpigot (`com.hpfxd.pandaspigot:pandaspigot-api:1.8.8-R0.1-SNAPSHOT`);
los plugins hechos para PandaSpigot/Paper 1.8.8 funcionan sin cambios.

## Créditos y licencia

VoidSoulSpigot se basa en el trabajo de:
- [PandaSpigot](https://github.com/hpfxd/PandaSpigot) (hpfxd)
- [Paper](https://github.com/PaperMC) (PaperMC)
- [Spigot / CraftBukkit / Bukkit](https://www.spigotmc.org/) (SpigotMC)

Ver [LICENSE.md](./LICENSE.md) para las licencias correspondientes.
