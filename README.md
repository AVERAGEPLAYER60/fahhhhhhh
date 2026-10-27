# Base Finder – port to Minecraft 26.3 (Fabric)

Source reconstructed from `basefinder-1_0_0.jar` (built for 1.21.4 / Yarn intermediary) by reading its
bytecode, then rewritten against Mojang's unobfuscated names for 26.3.

Behaviour kept 1:1: freecam (G), block ESP (H), chunk flags (J), menu (Right Shift), `/basefinder list|rescan`,
scoring weights, cluster logic, obsidian cap, config defaults.

## Build
Needs JDK 25. Check https://fabricmc.net/develop for the current Loom / Loader / Fabric API versions for 26.3
and adjust `gradle.properties` if needed, then:

    gradle wrapper --gradle-version latest
    ./gradlew build        # jar in build/libs/

## What changed and why
| Area | 1.21.4 | 26.3 |
|---|---|---|
| Build | `fabric-loom` + mappings | `net.fabricmc.fabric-loom`, no mappings, Java 25 |
| Names | Yarn/intermediary, refmap | Mojang names, no refmap |
| Keybinds | `KeyBindingHelper`, String category | `KeyMappingHelper`, `KeyMapping.Category`, `InputConstants.KEY_*` |
| Camera mixin | `Camera.update(BlockGetter,Entity,..)` | `Camera.update(DeltaTracker)` |
| Rendering | `RenderSystem` GL state + `Tessellator` + GL lines | custom `RenderPipeline` (no depth test) + `StagedVertexBuffer`; outlines are camera-facing quads |
| World render event | `WorldRenderEvents.LAST` | `LevelRenderEvents.AFTER_TRANSLUCENT_TERRAIN` |
| Screen | `render(DrawContext..)`, `client.setScreen` | `extractRenderState(GuiGraphicsExtractor..)`, `client.gui.setScreen` |
| Messages | action bar | chat (`sendSystemMessage`) |

## Not compiled against 26.3 – check these first if the build complains
I could verify the rendering, screen and key-mapping APIs against Fabric's 26.2 docs, but had no 26.3 jars to compile.
Most likely spots to need a one-line rename:
1. `KeyboardInputMixin` – `ClientInput.keyPresses` / `moveVector`, `Input.EMPTY`
2. `Freecam` – `Options.keyUp/keyDown/keyLeft/keyRight/keyJump/keyShift/keySprint`, `MultiPlayerGameMode.stopDestroyBlock()`
3. `CameraMixin` – `Camera.update(DeltaTracker)`, `setPosition`, `setRotation`, `isDetached`
4. `BaseFinderClient` – `ClientCommands` (was `ClientCommandManager`)
5. `BaseFinderRenderer` – 26.3 may have moved render APIs again; the docs pattern is 26.2
6. `MinecraftMixin` – `startAttack` / `startUseItem` / `continueAttack`
