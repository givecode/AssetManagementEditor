<p align="center">
  <img src="Image/AssetManagementEditor_Logo%20%282%29.png" alt="Asset Management Editor" width="720">
</p>

# Asset Management Editor for Unreal Engine 5 (UE5)

[简体中文](README_CN.md)

Asset Management Editor is an Unreal Engine editor plugin for repeatable Content Browser maintenance. It provides native batch tools for naming assets, organizing folders, generating Static Mesh LODs, and inspecting package references.

## Features

- Rename selected assets with class-specific prefix, suffix, and search/replace rules.
- Organize selected assets into class-named subfolders under their current folders.
- Apply a configurable LOD recipe to multiple Static Mesh assets.
- List package referencers and dependencies for the first selected asset.
- Use the tools from the Content Browser context menu or a dockable editor panel.
- Share project-owned naming rules through an `AssetNamingRuleData` Data Asset.
- Invoke rename, LOD, and reference-analysis operations from optional Editor Utility Blueprints.
- Protect mutating operations with confirmation dialogs, excluded directories, conflict checks, and editor transactions.

## Compatibility

| Item | Value |
| --- | --- |
| Plugin version | `1.1.2` |
| Source compatibility target | Unreal Engine `5.4`–`5.8` |
| Module | `AssetManagementEditor` |
| Module type | `Editor` |
| Loading phase | `Default` |
| C++ standard | C++20 |
| Required plugin | `EditorScriptingUtilities` |

The descriptor does not restrict target platforms. Install the plugin only for an Unreal Engine version shown as supported in Fab.

This is an editor-only plugin. Its tools and APIs are not available to packaged runtime builds.

## Install from Fab

1. Sign in to [Fab](https://www.fab.com/) with your Epic Games account and open the **Asset Management Editor** product page.
2. Purchase the plugin. If it is offered for free, add it to **My Library**.
3. Open **Epic Games Launcher > Unreal Engine > Library**.
4. Find **Asset Management Editor** in **Fab Library**, then click **Install to Engine**.
5. Select an Unreal Engine version that is installed on your computer and supported by the plugin, then complete the installation.
6. Open your project. If necessary, enable **Asset Management Editor** under **Edit > Plugins**.
7. Restart Unreal Editor if prompted.

If you encounter a `403` or another connection error during installation, check your network connection and confirm that Fab and Epic Games Launcher can access their online services.

## Quick Start

1. Open **Edit > Project Settings > Plugins > Asset Management Editor**.
2. Review the naming rules, confirmation setting, excluded directories, and Static Mesh LOD recipe.
3. Select one or more assets in the Content Browser.
4. Right-click the selection and open the **Asset Management** section, then choose an operation.
5. Alternatively, open **Window > Asset Management** to use the dockable panel.

Expected results:

- Mutating operations ask for confirmation by default.
- A result dialog reports succeeded, skipped, and failed items.
- Reference analysis appears in the Asset Management panel and does not modify assets.
- Modified Static Mesh packages are marked dirty and must still be saved by the user.

## Tools

### Rename Assets

The rename tool selects a rule using the asset's native class. Exact class matches take priority over parent-class matches. Rules from the optional `AssetNamingRuleData` asset are evaluated before the built-in settings rules.

For every supported asset, the tool:

1. Removes the first known prefix from the existing name, preferring longer prefixes.
2. Applies the rule's case-insensitive `SearchText` and `ReplacementText` transformation when configured.
3. Trims surrounding whitespace.
4. Adds the configured `Prefix` and `Suffix`.
5. Sanitizes the result as an Unreal object name and checks for conflicts before renaming.

Common built-in rules include:

| Asset class | Prefix |
| --- | --- |
| Blueprint | `BP_` |
| Static Mesh | `SM_` |
| Skeletal Mesh | `SK_` |
| Material | `M_` |
| Material Instance Constant | `MI_` |
| Texture 2D | `T_` |
| Animation Blueprint | `ABP_` |
| Animation Sequence | `AS_` |
| Niagara System | `NS_` |
| Level Sequence | `LS_` |
| Data Table | `DT_` |
| Input Action | `IA_` |

The complete default rule set is stored in [`Config/DefaultAssetManagementEditor.ini`](Config/DefaultAssetManagementEditor.ini).

### Custom Naming Rules

You can edit `BuiltInNamingRules` directly in Project Settings or create a reusable project-owned rule asset:

1. Create a Data Asset whose class is `AssetNamingRuleData`.
2. Add entries to its `Rules` array.
3. Assign the asset to **Naming Rule Data** in **Project Settings > Plugins > Asset Management Editor**.

Each rule supports:

- `AssetClass`
- `Prefix`
- `Suffix`
- `SearchText`
- `ReplacementText`

Project-owned Data Asset rules are evaluated first, so they can override a built-in rule for the same class.

### Organize by Asset Type

This operation creates a subfolder named after each asset's native class beneath its current package path and moves the asset into that folder. Assets already in the matching folder are skipped. Existing destination assets are treated as conflicts rather than overwritten.

Example:

```text
/Game/Environment/SM_Rock
→ /Game/Environment/StaticMesh/SM_Rock
```

### Apply Static Mesh LODs

The LOD tool operates only on Static Mesh assets and uses the recipe configured in Project Settings.

Default recipe:

| LOD | Percent triangles | Screen size |
| --- | ---: | ---: |
| LOD0 | `1.00` | `1.00` |
| LOD1 | `0.50` | `0.50` |
| LOD2 | `0.25` | `0.25` |

`bAutoComputeLODScreenSize` is disabled by default. Triangle percentages are clamped to `0.01`–`1.0`, and screen sizes are clamped to `0.0`–`1.0`.

Successful changes mark Static Mesh packages dirty but do not save them automatically.

### Analyze References

Reference analysis queries the Asset Registry for package-level referencers and dependencies of the first selected asset. Results are sorted and shown in the dockable panel or a dialog, depending on the entry point.

The report reflects the data currently available to the Asset Registry; it is not a runtime object-reference graph.

## Settings

Settings are available at **Edit > Project Settings > Plugins > Asset Management Editor**.

| Setting | Default | Purpose |
| --- | --- | --- |
| `NamingRuleData` | `None` | Optional project-owned rule Data Asset evaluated before built-in rules. |
| `BuiltInNamingRules` | Plugin rule set | Fallback class-based naming rules. |
| `bConfirmDestructiveOperations` | `True` | Confirms rename, organize, and LOD operations. |
| `ExcludedDirectories` | Empty | Package paths skipped by mutating operations; child paths are included. |
| `bAutoComputeLODScreenSize` | `False` | Lets Unreal Engine calculate LOD screen sizes when enabled. |
| `DefaultLODLevels` | Three levels | Reduction recipe including LOD0. |

On first load, plugin defaults are copied to the project's `Config/DefaultAssetManagementEditor.ini`. Configuration schema upgrades append missing default naming rules by asset class while retaining existing project rules.


## Safety and Limitations

- Rename, organize, and LOD operations modify editor assets. Use source control and review the selection before confirming.
- Excluded directories use package-path prefix matching.
- Assets without a matching naming rule are skipped.
- Naming and move conflicts are reported and are not overwritten.
- Organize by type uses the native asset class name as the destination folder name.
- LOD generation supports Static Mesh assets only and does not save modified packages automatically.
- Reference analysis processes only the first selected asset and reports package dependencies known to the Asset Registry.
- Built-in rules that reference optional engine or project plugins only apply when those asset classes are available.
- The plugin is editor-only and should not be called from runtime gameplay code.

## Repository Layout

```text
AssetManagementEditor/
├─ AssetManagementEditor.uplugin
├─ Config/
├─ Resources/
├─ Source/AssetManagementEditor/
│  ├─ Public/
│  └─ Private/
└─ Changelog/
```

## Support

- Issue tracker: [GitHub Issues](https://github.com/givecode/AssetManagementEditor/issues)

When reporting a build problem, include the Unreal Engine version, target platform, complete compiler error, and the plugin version.

## Copyright

Copyright (c) 2026 haozena. All Rights Reserved.

