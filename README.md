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
    - `chunks.maxLightGapChecksPerTick`: máximo de columnas de luz de cielo recalculadas por
      chunk y por tick (default `0` = vanilla). Cuando un bloque cambia a la altura del bloque
      más alto de su columna o por encima, esa columna se marca para recalcular la luz de cielo,
      y el chunk se las come **todas de golpe en un solo tick**. Un reset de mina a cielo abierto
      marca las 256 columnas de cada chunk que toca — eso es lo que convierte el reset en un pico
      de TPS. Bajarlo reparte ese trabajo entre los ticks siguientes. Sólo aplica a minas
      abiertas al cielo.
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

- **Protección contra paquetes maliciosos**:
    - Validación de slot en `PacketPlayInWindowClick`: el slot es un `short` que venía del
      cliente sin cota superior, y varios modos de click indexaban la lista de slots con él
      sin comprobar. Un paquete manipulado hacía saltar `IndexOutOfBoundsException` en el hilo
      principal, y cada excepción se registraba con stack trace completo → spam de log y disco.
    - Validación del índice de botón en `PacketPlayInEnchantItem`, que indexaba un array de 3
      posiciones directamente con el valor del cliente.
    - Contención genérica de errores de paquete: las excepciones de cualquier handler dejan de
      escaparse a `SystemUtils` (que loggeaba en FATAL una por paquete). Ahora se agrupan en una
      entrada cada 10 s con el número de repeticiones suprimidas, y `security.kickOnPacketError`
      (desactivado por defecto) expulsa a quien las provoca. Esto también cubre bugs de handler
      que todavía no se han encontrado.

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
      maxLightGapChecksPerTick: 32 # reparte el recálculo de luz tras un reset de mina
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

### Ajustes fuera de `voidsoul.yml`

A estas alturas la mayor parte del margen que queda en el servidor está en estas opciones, no en
más parches. Ordenadas por impacto para un prison:

| Archivo | Opción | Por qué |
| --- | --- | --- |
| `spigot.yml` | `anti-xray.enabled: false` | Activo por defecto. Cuesta CPU en cada envío de chunk y en cada cambio de bloque — justo lo que más hace una mina al resetearse. Si lo necesitas, `engine-mode: 1` es más barato que el 2. |
| `bukkit.yml` | `spawn-limits.*: 0` | Si no hay mobs en el prison, el intento de spawn por chunk desaparece entero. |
| `server.properties` | `view-distance` | Bajarlo de 10 a 5-6 reduce a la cuarta parte los chunks que se envían y se tickean por jugador. Es probablemente el ajuste con mejor relación impacto/esfuerzo. |
| por mundo | `/gamerule randomTickSpeed 0` | En las minas nada necesita random ticks. Quita el sorteo de 3 bloques por sección y por chunk cercano a jugadores. |
| `spigot.yml` | `merge-radius.item`, `item-despawn-rate` | Complementan a `maxItemsPerChunk`. |
| `spigot.yml` | `max-entity-collisions`, `entity-activation-range`, `hopper-transfer`, `hopper-check` | |
| `paper.yml` | `disable-thunder`, `disable-ice-and-snow`, `disable-mood-sounds` | Corren por cada chunk cercano a un jugador y en un prison no aportan nada. |
| `paper.yml` | `tick-next-tick-list-cap` | Limita la avalancha de updates programados si tus minas tienen arena, grava o líquidos. |
| `paper.yml` | `keep-spawn-loaded` | |

### Lo que ya está optimizado (no busques ahí)

Al auditarlo comprobé que estas rutas ya están cubiertas por PandaSpigot/Paper, por si te ahorra
tiempo: el ticking de chunks ya se limita a los chunks cercanos a jugadores (así que los random
ticks, el clima y el hielo **no** corren en las minas vacías), la carga de chunks ya es asíncrona
(`ChunkIOExecutor`), las explosiones ya cachean densidad (útil para picos explosivos), la lista de
tile entities ya contiene sólo las que tickean, y ya existe caché de `ChunkMap` para el envío de
chunks.

### El reset de minas: lo que el fork no puede arreglar solo

Si tras todo esto los resets siguen dando picos, el cuello de botella está en **cómo escribe los
bloques tu plugin de minas**, y eso no lo puede optimizar el servidor. La diferencia es grande:

- `block.setType(...)` bloque a bloque paga por cada bloque la actualización de heightmap, el
  marcado de luz y la comprobación de tile entity.
- Escribir directo en las `ChunkSection`, llamar a `initLighting()` una vez por chunk al final y
  reenviar los chunks afectados es órdenes de magnitud más rápido.

Si tu plugin permite elegir, activa el modo "rápido"/NMS y **escalona** los resets (que no
caigan todas las minas en el mismo tick).

## Seguridad

Ya venían cubiertos por PandaSpigot/Paper y no hizo falta tocarlos: Log4Shell (log4j 2.25.3),
límites de tamaño de libros y carteles, cooldown de edición/firma de libros, límite de NBT en
paquetes (50 KB), rechazo de movimiento con NaN/Infinity, límite de 128 canales de plugin,
`filterCreativeItems`, y el limitador de paquetes de `pandaspigot.yml`.

Lo que **no** puede arreglar el fork y depende de tu configuración:

- Si usas `bungeecord: true` en `spigot.yml`, el servidor confía en la IP y el UUID que le manda
  el proxy. **Tiene que estar cortafuegos de por medio**: si el puerto es accesible desde fuera,
  cualquiera puede conectarse suplantando a cualquier jugador, incluidos los admins.
- `online-mode=false` sin proxy tiene el mismo problema.
- Los exploits de plugins (dupes de economía, GUIs de venta, `/pay` con cantidades negativas) van
  por su cuenta; el fork no puede verlos.

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
