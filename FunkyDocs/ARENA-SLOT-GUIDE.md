# Arena Slot Loader - creator guide 

# WIP DOCS 
> Images needed also remember to delete this section when posting

**Version 1.0.0**

Arena Slot Loader adds your arena to Infinite Disco's arena selector alongside the
stock arenas and other installed arena mods. Players can install several compatible
arenas together and choose between them. All their slots remain available; the
loader streams in the map belonging to the selected slot.

You can use the Unreal project you already make arena mods in. Your mod needs its
own cooked map and dependencies, an `info.json`, and optionally a cooked selection
image. The examples in this guide can be copied directly.

## TLDR

Already have a working arena? Here's the short version. Replace `YourName` and
`MyArena` with your own names.

1. [Give your map its own folder](#convert-an-existing-arena):
   `Content/ArenaMods/YourName/MyArena/`. Move or duplicate it in Unreal's Content
   Browser and fix redirectors after moving assets.
2. [Add a selection image](#add-an-optional-selection-image) if you want one.
   For different looks in the same map, [add arena styles](#add-optional-arena-styles).
3. [Cook your map](#cook-and-package-the-arena), its dependencies and the optional
   image with **`-VersionCookedContent`**.
4. [Make an info.json](#write-infojson) using the example below. Give each arena its
   own ID and map path. Add `ArenaThumbnail` if you're using a custom image,
   or `ArenaStyles` for a map with selectable data-layer variations.
5. [Put the JSON and all three archive files together](#package-the-player-download)
   in your own folder under `~mods`, as shown below.
6. With Arena Slot Loader installed, check that your slot loads, a song starts and
   enemies spawn. Switch to another arena and back too.

Example `info.json` without a custom image:

```json
{
  "ArenaName": "My Arena",
  "ArenaId": "Environment.Arena.Mod.YourName.MyArena",
  "ArenaMap": "/Game/ArenaMods/YourName/MyArena/LI_Arena_MyArena"
}
```

Finished mod folder:

```text
Pagoda/Content/Paks/~mods/YourName_MyArena/
  info.json
  YourName_MyArena_P.pak
  YourName_MyArena_P.utoc
  YourName_MyArena_P.ucas
```

## Contents

- [TLDR](#tldr)
- [How an arena becomes a slot](#how-an-arena-becomes-a-slot)
- [Before you begin](#before-you-begin)
- [Choose unique names and paths](#choose-unique-names-and-paths)
- [Convert an existing arena](#convert-an-existing-arena)
- [Add an optional selection image](#add-an-optional-selection-image)
- [Add optional arena styles](#add-optional-arena-styles)
- [Include your content in the cook](#include-your-content-in-the-cook)
- [Cook and package the arena](#cook-and-package-the-arena)
- [Write info.json](#write-infojson)
- [Package the player download](#package-the-player-download)
- [Support several installed arenas](#support-several-installed-arenas)
- [Test before releasing](#test-before-releasing)
- [Troubleshooting](#troubleshooting)

## How an arena becomes a slot

The game mounts your cooked archives when it starts. Arena Slot Loader reads the
`info.json` files inside `Pagoda/Content/Paks/~mods`, then adds an entry for each
valid, uniquely identified arena to the native catalogue. It uses the name and
optional image from the JSON to describe that entry.

When a player selects your entry, the loader uses `ArenaMap` to load your map into
the native Infinite Disco arena instance. 

**You do not need to create an arena-definition Data Asset.** The shared loader
constructs that menu information at runtime. Unreal assets are still used for the
map, meshes, materials and optional texture. An optional `PrimaryAssetLabel` used
for cooking has a different purpose: it tells Unreal which content to cook and
package. It does not create the selector entry. Woahhh technology am I right?

## Before you begin

You need:

- A compatible **Pagoda Unreal project** and its working arena cooking/packaging
  setup. The current validation baseline is **Unreal Engine 5.7.4**. Keep the engine version and project settings that match the game
  build you target.
- The editable source of your arena, or assets for a new arena. Cooked `.pak`,
  `.utoc` and `.ucas` files are distribution output, not a replacement for the
  editable Unreal project. You should know this, but some people may, probably will, no actually they will try this with cooked assets.
- Arena Slot Loader 1.0.0 and a compatible experimental UE4SS installed in the
  retail game for testing.

If you already have an arena project, keep its plugins, source, render settings and
cooker.
The slot format works with standard Unreal assets in a Blueprint-only project;
your project's existing plugins may have their own requirements.

For your first arena, start with a compatible community **Pagoda modding project**.
The [community arena tutorial](https://docs.google.com/document/d/1XFfOpgofzVanfc0RoQGdQtYiRDEpmmXJVLCMmQWsSPA/edit)
identifies the Pagoda template and its Default arena reference. That tutorial
assumes the community's basic modding-project setup has already been completed.

Back up the source project before moving assets. Work in Unreal's Content Browser
so its references are updated along with the assets. Tiny disclaimer so I am not to blame when it all goes wrong.

## Choose unique names and paths

Use your own creator name and arena name in place of `YourName` and `MyArena`
throughout this guide. A suitable layout is:

```text
Content/
  ArenaMods/
    YourName/
      MyArena/
        LI_Arena_MyArena.umap
        T_ArenaSelection.uasset    Optional
        PAL_MyArena.uasset         Optional cook label
        Meshes/
        Materials/
        Textures/
```

In Unreal's Content Browser these appear as assets without their file extensions.
Use letters, digits and underscores for the folder and asset names used in your
manifest paths. Reserve spaces and punctuation for the display name.

| Item | Example | Purpose |
|---|---|---|
| Display name | `My Arena` | Text players see in the selector |
| Arena ID | `Environment.Arena.Mod.YourName.MyArena` | Stable identity for this slot |
| Map package | `/Game/ArenaMods/YourName/MyArena/LI_Arena_MyArena` | Map the loader streams |
| Image object | `/Game/ArenaMods/YourName/MyArena/T_ArenaSelection.T_ArenaSelection` | Optional texture for the selector |
| Install folder | `YourName_MyArena` | Keeps the manifest and archives together |

`/Game` means the Unreal project's `Content` folder. The map and optional image
named in `info.json` must be under `/Game/ArenaMods/` or the older `/Game/Mods/`
path. Use `ArenaMods` for new arenas. Existing `/Game/Mods/` arenas still work and
do not need moving. Avoid the loader's own
`/Game/Mods/ArenaSlotLoader/` folder. Give your dependencies unique package paths
too; existing unique dependency paths can remain elsewhere if your packaging
includes them.

The map's filename does not have to begin with `LI_`; that is a naming convention.
The manifest must match the actual cooked package path exactly.

## Convert an existing arena

Use this route if the map already works as a normal arena replacement.

1. Open a backed-up copy of the working project. Find the actual level used by
   your replacement, commonly `LI_Arena_Default` under the game's Default folder.
2. Create your unique folder under `Content/ArenaMods/YourName/MyArena`.
3. Duplicate the working level in the Content Browser, move the duplicate into
   that folder, and rename it `LI_Arena_MyArena`. Retain the original as your
   reference, but exclude its stock-path replacement from the slot package.
4. Inspect the new map's references. Move or duplicate custom dependencies that
   currently overwrite stock assets into unique paths, and update the map to use
   them. Keep references to genuine unchanged base-game assets where appropriate.
5. After moves or renames, fix redirectors in the affected folders. Save all
   affected assets, close and reopen the map, and verify its meshes and materials.
   Unreal's [redirector documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/asset-redirectors-in-unreal-engine)
   explains how this updates references to the moved packages.
6. Update the project's cook label or explicit cook list to include the new map
   and its dependencies. Exclude the old replacement map, old replacement images
   and any other stock-path overrides from this arena's output.

For maps with sublevels, level instances or external actors, inspect those
dependencies too. Duplicating the top-level map does not automatically give every
referenced asset a unique path. Use the Reference Viewer and your packager's
content listing to inspect what the final mod contains.

Renaming the install folder or archive files alone is insufficient. A map still
stored inside the archives at the stock Default path remains a replacement.

Continue with the optional image, cooking and manifest steps below.

## Add an optional selection image

Import your artwork into your arena folder with Unreal's **Import** button. The
resulting asset must be a **Texture2D**. Name it `T_ArenaSelection`, for example.
Use your project's normal UI texture settings and inspect the framing against the
arena selector's title overlay in game. The loader does not prescribe an image
resolution. I am lazy. Sorry.

Set `ArenaThumbnail` in `info.json` to the cooked texture's object path:

```text
/Game/ArenaMods/YourName/MyArena/T_ArenaSelection.T_ArenaSelection
```

This supplies the native arena selection image. Package the texture
with your own mod, under your own path, so every installed arena can have its own
image.

A loose PNG next to the archives is not used by this field. The imported texture
must be cooked and included in the archives. Omit `ArenaThumbnail` if you want the
native fallback image.

## Add optional arena styles

Day and night versions of the same arena can share one map and one `info.json`.
Each entry in `ArenaStyles` gets a name and optional image in the native selector.
Selecting a different style switches its Runtime data layer without reloading the
map. If your arena only has one look, skip this section; the basic JSON still works.

Use your existing project's working Runtime data-layer setup. This needs a map
that supports runtime data-layer streaming (World Partition), not just editor
folders or ordinary actor layers. Adding the JSON does not convert a map or turn
editor-only layers into runtime layers. If you are converting an existing arena,
check that its layers actually switch in play before packaging it.

1. In your arena's folder, create a **Data Layer Asset** for each style and set its
   **Data Layer Type** to **Runtime**. For example, `DL_Day` and `DL_Night`.
   Existing Runtime assets can be reused if they have your own unique paths.
2. Open **Window → World Partition → Data Layers Outliner**. Add those assets as
   data-layer instances in your arena map and assign the actors for each look to
   the matching instance. Merely creating the assets in the Content Browser does
   not assign any actors. Keep each style's actors out of the other style layers.
3. Leave shared floors, collision and other common actors outside the style layers,
   or put them in a separate Runtime layer such as `DL_Common` and declare it as
   `ArenaBaseLayer`. The loader keeps that layer active for every style.
4. Keep the declared style layers at the root, or make them children of your
   declared common layer. Do not put a style beneath another style or an inactive
   undeclared parent. Start the common layer as **Activated** and style layers as
   **Unloaded**. Any additional children you want with a style should start as
   **Activated**, so their parent's state controls them.
5. Save the map, data-layer assets and external actors. Include them in the cook,
   along with every style's dependencies and optional image.

This example gives one arena two styles:

```json
{
  "ArenaName": "My Arena",
  "ArenaId": "Environment.Arena.Mod.YourName.MyArena",
  "ArenaMap": "/Game/ArenaMods/YourName/MyArena/LI_Arena_MyArena",
  "ArenaThumbnail": "/Game/ArenaMods/YourName/MyArena/T_ArenaSelection.T_ArenaSelection",
  "ArenaBaseLayer": "/Game/ArenaMods/YourName/MyArena/DL_Common.DL_Common",
  "ArenaStyles": [
    {
      "Name": "Day",
      "DataLayer": "/Game/ArenaMods/YourName/MyArena/DL_Day.DL_Day"
    },
    {
      "Name": "Night",
      "DataLayer": "/Game/ArenaMods/YourName/MyArena/DL_Night.DL_Night",
      "Thumbnail": "/Game/ArenaMods/YourName/MyArena/T_Night.T_Night"
    }
  ]
}
```

`ArenaBaseLayer` is optional; remove it if your common actors are not in a layer.
Every style needs a nonempty `Name` and its own `DataLayer` object path. A style's
`Thumbnail` overrides `ArenaThumbnail`; leaving both out uses the game's fallback.
Use the same package.object format as textures, without `.uasset`.

Keep style names and layer paths distinct, including from the common layer. The
array order is the style order; keep it stable across updates because the game
remembers a numeric style selection. You do not need a separate map, arena ID or
arena-definition asset for each look. Do not combine `ArenaStyles` with the old
`ArenaDefinition` field.

The loader activates the common layer and selected style, and unloads the other
declared styles. It does not run a custom material-swap Blueprint or expand the
playable area. Test every style with a song running, then change arenas and return.
Check that the old look and its collision disappear, the selected image is right,
and Charlie's entrance, music and enemies still work.

Epic's [data layers guide](https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition---data-layers-in-unreal-engine?application_version=5.7)
covers the editor setup and the difference between Editor and Runtime layers.

## Include your content in the cook

Use the asset list, map selection or cook labels already supported by your project
and packager. Include the new map, all its required custom dependencies, and the
optional selection texture. For styled arenas, include the Runtime data-layer
assets, external actors and every style's textures and dependencies too.

**JSON references do not cause Unreal to cook an asset.** In particular, the image
can be absent from the output even though its path in `info.json` is correct. Add it
explicitly to the cook selection or to a cook label.

If your normal setup uses labels:

1. In your arena folder, choose **Add / right-click → Miscellaneous → Data Asset**.
2. Select Unreal's built-in **PrimaryAssetLabel** class and name the asset
   `PAL_MyArena`.
3. Set **Rules → Cook Rule** to **Always Cook**. Use your project's usual chunk ID
   and priority rules; avoid changing the project's existing chunk scheme blindly.
4. Enable **Label Assets in My Directory** if you want the folder's content included.
   Add your map, optional image and any style data-layer assets/images to
   **Explicit Assets** so their inclusion is clear.
5. Save the label. Confirm that your project's Asset Manager/cooker discovers it,
   and inspect the cooked/package output to verify the requested assets were included.

This is the standard Unreal label described in Epic's
[asset chunking guide](https://dev.epicgames.com/documentation/en-us/unreal-engine/preparing-assets-for-chunking-in-unreal-engine).
The label is a cooking aid. It is optional when your existing cooker explicitly
includes the same content; it is not an arena-definition asset and does not replace
`info.json`.

## Cook and package the arena

Something something plugin when I re-release it 

## Write info.json

Create a plain UTF-8 text file named exactly `info.json`. For an arena without a
custom image, copy and edit this complete example:

```json
{
  "ArenaName": "My Arena",
  "ArenaId": "Environment.Arena.Mod.YourName.MyArena",
  "ArenaMap": "/Game/ArenaMods/YourName/MyArena/LI_Arena_MyArena"
}
```

For an arena with a selection image, use:

```json
{
  "ArenaName": "My Arena",
  "ArenaId": "Environment.Arena.Mod.YourName.MyArena",
  "ArenaMap": "/Game/ArenaMods/YourName/MyArena/LI_Arena_MyArena",
  "ArenaThumbnail": "/Game/ArenaMods/YourName/MyArena/T_ArenaSelection.T_ArenaSelection"
}
```

| Field | Required | Rules |
|---|---|---|
| `ArenaName` | Yes | Nonempty display name; spaces are allowed. |
| `ArenaId` | Yes | Unique ID beginning `Environment.Arena.Mod.`, followed by creator and arena segments. Use letters, digits and underscores within each segment. Keep it stable across updates. |
| `ArenaMap` | Yes | Unique `/Game/ArenaMods/` map package path; no extension or object suffix. Older `/Game/Mods/` paths also work. |
| `ArenaThumbnail` | No | `/Game/ArenaMods/` Texture2D object path; the name after the dot must match the final package name. Omit the field when unused. Older `/Game/Mods/` paths also work. |
| `ArenaStyles` | No | Nonempty array of styles for the same map. Each has `Name`, `DataLayer` and optional `Thumbnail`; see the styles section. |
| `ArenaBaseLayer` | No | Runtime DataLayerAsset object path for common content. Used with `ArenaStyles`; must differ from the style layers. |

If you use Unreal's **Copy Reference**, remove the class name and surrounding quotes.
For example:

```text
World'/Game/ArenaMods/YourName/MyArena/LI_Arena_MyArena.LI_Arena_MyArena'
```

becomes this `ArenaMap` value:

```text
/Game/ArenaMods/YourName/MyArena/LI_Arena_MyArena
```

Keep the `.T_ArenaSelection` object suffix for the texture. Do not include `.umap`
or `.uasset` in either field.

Use the field names exactly as shown, double quotes, forward slashes and valid JSON
without comments or trailing commas. Enable filename extensions in Windows and
check that the file is not named `info.json.txt`. You do not need to add the ID to
the project's gameplay-tag dictionary.

One manifest describes one slot. Do not put an array of arenas in a single
`info.json`. `ArenaStyles` is an array of looks within that one arena. See the
multiple-arena layout below for a collection of different maps.

## Package the player download

Place the manifest and complete archive set together in a unique install folder:

```text
Pagoda/
  Content/
    Paks/
      ~mods/
        YourName_MyArena/
          info.json
          YourName_MyArena_P.pak
          YourName_MyArena_P.utoc
          YourName_MyArena_P.ucas
```

Use the same base name for all companion archives. Zip the top-level `Pagoda`
folder so players can extract it into their Dead as Disco game root. They should
end up with the structure above, not an extra nested `Pagoda/Pagoda` directory.

List **Arena Slot Loader 1.0.0** and compatible experimental **UE4SS** as requirements
on your mod page. Your arena download contains your content and manifest; players
install the shared loader separately.

Suggested installation text for your mod page:

> Install Arena Slot Loader and its UE4SS prerequisite. With the game closed,
> extract this download's Pagoda folder into your Dead as Disco game folder.
> Start the game and choose this arena in the Infinite Disco arena selector.

When updating, keep the same arena ID and replace the complete archive set together.
If archive filenames change, tell players to remove the obsolete set from this
arena's folder first. Keep unrelated mods intact. Restart the game after installing,
updating or removing an arena; live installation is not the supported workflow.

## Support several installed arenas

Every arena needs its own ID and map package path. The loader registers all valid
installed entries together. For example, a creator can distribute a collection as:

```text
Pagoda/Content/Paks/~mods/
  YourName_Rooftop/
    info.json
    YourName_Rooftop_P.pak
    YourName_Rooftop_P.utoc
    YourName_Rooftop_P.ucas
  YourName_Warehouse/
    info.json
    YourName_Warehouse_P.pak
    YourName_Warehouse_P.utoc
    YourName_Warehouse_P.ucas
```

| Entry | Rooftop | Warehouse |
|---|---|---|
| `ArenaName` | `Rooftop` | `Warehouse` |
| `ArenaId` | `Environment.Arena.Mod.YourName.Rooftop` | `Environment.Arena.Mod.YourName.Warehouse` |
| `ArenaMap` | `/Game/ArenaMods/YourName/Rooftop/LI_Rooftop` | `/Game/ArenaMods/YourName/Warehouse/LI_Warehouse` |

Both slots appear alongside other installed arenas. Selecting Warehouse switches
the active map; it does not remove Rooftop's slot. Each arena can have its own image.

Keep custom dependencies under unique paths too. Renaming only an archive does not
prevent two mods from overwriting the same internal asset. The loader rejects
duplicate arena IDs or map paths, including names that differ only in letter case;
do not rely on load order to choose which duplicate wins.

Remove the old Default-replacement edition of a converted mod from the test game.
It would still overwrite the stock arena independently of the new slot. Sharing
the native arena instance while switching slots does not require every creator to
package a replacement of `LI_Arena_Default`.

## Test before releasing

I do not even think I will explain this part. Please do not release untested stuff that just makes everybody look bad.

## Troubleshooting

| Symptom | What to check |
|---|---|
| No custom slots appear | UE4SS and the current loader are running; BPModLoaderMod is enabled; archives and manifests are in the correct folders. Check the discovery and registration messages. |
| One arena is missing | Its JSON is valid; ID and map are unique; the log does not report a rejected manifest. An unrelated mod's `info.json` does not become an arena entry. |
| JSON or path rejected | Exact field names; no comments/trailing commas; `/Game/ArenaMods/` or `/Game/Mods/` prefix; valid characters; map without object suffix; image with matching object suffix. |
| Image falls back | The image is a cooked Texture2D at the exact path; it was explicitly included in the cook and archives. |
| A styled arena is missing | Check the registration log for a missing or non-Runtime DataLayerAsset; cook every declared layer at its exact path. |
| Style name changes but the look does not | The map must support runtime data-layer streaming; the assets need instances and assigned actors in that map. Check inactive parents, overlapping assignments and the loader's style-change log. |
| Styles overlap or leave collision behind | Keep style-specific actors out of the common layer and other styles; check actor references that keep content loaded and any separate collision actors. |
| Black or empty arena | The exact map and dependencies are packaged; the cook used versioned content; materials have compatible shaders; required actors are in an active data layer. |
| Wrong height, blocked movement or overlapping floors | Compare the converted map with its working source; check reference-floor collision and accidental transform changes; remove any old Default replacement from the test installation. |
| Map loads but no song or enemies | Confirm the current loader runtime and matching LogicMods files are installed, then inspect challenge/intro events in the logs. Adding music or spawn scripts to the arena is not the standard fix. |
| Enemies spawn but cannot move | Inspect collision, obstacles and navigation relative to the native gameplay area. Slot loading does not expand that area. |
| One mod changes another | Check for shared internal package paths and stock overrides; changing archive filenames alone cannot isolate those assets. |
| Changes do not appear | Close the game, replace all companion archives from the same new build, remove obsolete copies of this mod and restart. |
| A previously working mod fails after a game update | Compare the game/engine build and loader compatibility, then recook as needed. Preserve logs so a content problem can be distinguished from a loader problem. |

If this works, wohoo. If it doesn't more hours in the pits for me.
