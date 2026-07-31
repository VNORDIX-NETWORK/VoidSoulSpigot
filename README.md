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
    - `chunks.unloadDelayTicks`: ticks que un chunk espera en la cola antes de descargarse
      de verdad (default `0` = vanilla). Si el jugador vuelve antes de que expire, el chunk
      sale de la cola sin ninguna E/S de disco. Pensado para servidores donde la gente salta
      constantemente entre zonas (minas privadas, plots, arenas).
    - Toggles de tick por mundo: `ticking.tickHoppers`, `tickFurnaces`, `tickBrewingStands`,
      `tickBeacons`, `tickVillages`.
    - `ticking.tileEntityActivationRange`: sólo tickea tile entities (hornos, hoppers, alambiques,
      beacons, spawners…) a menos de N bloques de un jugador (default `0` = vanilla). El conjunto
      de chunks activos se recalcula una vez por tick, así que la comprobación por tile entity es
      un único lookup de hash.
- **Entidades** (sección `entities`):
    - `playerCollisions`: si es `false`, los jugadores dejan de empujar entidades. Elimina el
      escaneo de entidades cercanas que cada jugador hace cada tick — el coste más alto cuando
      se amontona gente en la entrada de una mina o en el spawn — y quita el empuje jugador
      contra jugador. Los mobs siguen pudiendo empujar a los jugadores.
    - `maxItemsPerChunk`: máximo de ítems tirados por chunk (default `0` = sin límite). Al pasarse,
      se elimina el ítem más antiguo del chunk. Red de seguridad contra avalanchas de drops.
    - `itemMergeCheckIntervalTicks`: ticks mínimos entre escaneos de fusión de ítems
      (default `0` = comportamiento Spigot). Los ítems se fusionan un poco más tarde pero el
      número de escaneos baja muchísimo.

Todo lo de PandaSpigot sigue disponible: Netty 4.1, tick loop moderno, knockback base, eventos backporteados
(`PlayerChunkLoadEvent`, `EntityMoveEvent`, etc.), Adventure API y más. Ver [patches](./patches/).

## Compilar

Requisitos: **JDK 17+** (probado con JDK 21), **Git** y **Bash** (en Windows, Git Bash).

```bash
./panda jar
```

El jar final queda en `VoidSoulSpigot-paperclip.jar`.

## Perfil recomendado para prison (minas privadas + PvP)

Todas las opciones nuevas vienen desactivadas por defecto para no cambiar el comportamiento
vanilla sin que lo pidas. Punto de partida sugerido en `voidsoul.yml`:

```yaml
worlds:
  default:
    chunks:
      maxUnloadsPerTick: 50
      maxAutosaveChunksPerPass: 200
      unloadDelayTicks: 400        # ~20 s; evita descargar y recargar minas al saltar entre ellas
    ticking:
      tileEntityActivationRange: 96
    entities:
      playerCollisions: false      # sin empujones entre jugadores
      maxItemsPerChunk: 300
      itemMergeCheckIntervalTicks: 10
```

Ten en cuenta los efectos de juego antes de aplicarlo:

- `tileEntityActivationRange` congela hornos y hoppers mientras no haya nadie cerca. Si vendes
  con hoppers automáticos sin jugadores presentes, súbelo o déjalo en `0`.
- `playerCollisions: false` quita el empuje jugador contra jugador en todo el mundo. Si tu PvP
  depende de empujar al rival hacia un borde, no lo actives en el mundo de la arena.
- `maxItemsPerChunk` **borra** ítems cuando se pasa del límite. 300 es holgado para minar, pero
  si tienes zonas donde se acumulan drops a propósito, súbelo.

Fuera de `voidsoul.yml`, para este tipo de servidor conviene revisar también:

- `spigot.yml` → `anti-xray.enabled`: está activo por defecto y cuesta CPU en cada envío de chunk
  y en cada cambio de bloque, justo lo que más hace una mina al resetearse.
- `spigot.yml` → `merge-radius.item` y `item-despawn-rate`: complementan a `maxItemsPerChunk`.
- `spigot.yml` → `max-entity-collisions`, `entity-activation-range`, `hopper-transfer` /
  `hopper-check`.
- `paper.yml` → `keep-spawn-loaded`, `use-async-lighting`.

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
