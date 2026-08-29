<p align="center">
  <img src="Image/AssetManagementEditor_Logo%20%282%29.png" alt="Asset Management Editor" width="720">
</p>

# Asset Management Editor for Unreal Engine 5 (UE5)

[English](README.md)

Asset Management Editor 是一个用于标准化 Content Browser 维护流程的虚幻引擎编辑器插件。它提供原生批处理工具，用于资产命名、目录整理、Static Mesh LOD 生成以及包引用检查。

## 功能特性

- 按资产类规则批量添加前缀、后缀并执行查找替换。
- 将所选资产整理到当前目录下以资产类命名的子目录中。
- 为多个 Static Mesh 资产应用可配置的 LOD 方案。
- 列出第一个所选资产的包引用者与依赖项。
- 可通过 Content Browser 右键菜单或可停靠编辑器面板使用工具。
- 通过 `AssetNamingRuleData` Data Asset 共享项目自有命名规则。
- 可从 Editor Utility Blueprint 调用重命名、LOD 和引用分析操作。
- 通过确认对话框、排除目录、冲突检查和编辑器事务保护修改操作。

## 兼容性

| 项目 | 值 |
| --- | --- |
| 插件版本 | `1.1.2` |
| 源码兼容目标 | Unreal Engine `5.4`–`5.8` |
| 模块 | `AssetManagementEditor` |
| 模块类型 | `Editor` |
| 加载阶段 | `Default` |
| C++ 标准 | C++20 |
| 必需插件 | `EditorScriptingUtilities` |

描述符没有限制目标平台。请只为 Fab 中显示为受支持的虚幻引擎版本安装插件。

这是一个仅编辑器插件，其工具和 API 不会出现在打包后的运行时版本中。

## 从 Fab 安装

1. 使用 Epic Games 账号登录 [Fab](https://www.fab.com/)，打开 **Asset Management Editor** 商品页面。
2. 购买插件；如果商品免费，则将其添加到 **My Library**。
3. 打开 **Epic Games Launcher > Unreal Engine > Library**。
4. 在 **Fab Library** 中找到 **Asset Management Editor**，点击 **Install to Engine**。
5. 选择一个本机已经安装且受插件支持的 Unreal Engine 版本并完成安装。
6. 打开项目；如果需要，请在 **Edit > Plugins** 中启用 **Asset Management Editor**。
7. 如果编辑器提示重启，请重启 Unreal Editor。

如果安装过程中出现 `403` 或其他连接错误，请检查网络连接，并确认 Fab 和 Epic Games Launcher 能够正常访问在线服务。

## 快速开始

1. 打开 **Edit > Project Settings > Plugins > Asset Management Editor**。
2. 检查命名规则、确认选项、排除目录和 Static Mesh LOD 方案。
3. 在 Content Browser 中选择一个或多个资产。
4. 右键单击所选资产，在 **Asset Management** 区域中选择操作。
5. 也可以通过 **Window > Asset Management** 打开可停靠面板。

预期结果：

- 默认情况下，修改类操作会先请求确认。
- 结果对话框会报告成功、跳过和失败的项目数。
- 引用分析显示在 Asset Management 面板中，不会修改资产。
- 修改后的 Static Mesh 包会被标记为未保存，仍需用户手动保存。

## 工具说明

### Rename Assets

重命名工具根据资产的原生类选择规则。精确类匹配优先于父类匹配；可选 `AssetNamingRuleData` 资产中的规则先于设置内置规则执行。

对于每个受支持的资产，工具会：

1. 从现有名称中移除第一个已知前缀，并优先匹配更长的前缀。
2. 如果已配置，则执行不区分大小写的 `SearchText` 与 `ReplacementText` 转换。
3. 去除首尾空白。
4. 添加配置的 `Prefix` 和 `Suffix`。
5. 将结果清理为有效的 Unreal 对象名称，并在重命名前检查冲突。

常用内置规则包括：

| 资产类 | 前缀 |
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

完整默认规则位于 [`Config/DefaultAssetManagementEditor.ini`](Config/DefaultAssetManagementEditor.ini)。

### 自定义命名规则

你可以直接在 Project Settings 中编辑 `BuiltInNamingRules`，也可以创建可复用的项目自有规则资产：

1. 创建类为 `AssetNamingRuleData` 的 Data Asset。
2. 在其 `Rules` 数组中添加规则。
3. 在 **Project Settings > Plugins > Asset Management Editor** 中将该资产指定给 **Naming Rule Data**。

每条规则支持：

- `AssetClass`
- `Prefix`
- `Suffix`
- `SearchText`
- `ReplacementText`

项目自有 Data Asset 规则优先执行，因此可以覆盖同一资产类的内置规则。

### Organize by Asset Type

此操作会在每个资产当前包路径下创建以原生资产类命名的子目录，并将资产移动到该目录。已经位于对应目录的资产会被跳过；如果目标位置已经存在资产，则会报告冲突而不是覆盖。

示例：

```text
/Game/Environment/SM_Rock
→ /Game/Environment/StaticMesh/SM_Rock
```

### Apply Static Mesh LODs

LOD 工具仅处理 Static Mesh 资产，并使用 Project Settings 中配置的方案。

默认方案：

| LOD | 三角形比例 | Screen Size |
| --- | ---: | ---: |
| LOD0 | `1.00` | `1.00` |
| LOD1 | `0.50` | `0.50` |
| LOD2 | `0.25` | `0.25` |

`bAutoComputeLODScreenSize` 默认关闭。三角形比例会限制在 `0.01`–`1.0`，Screen Size 会限制在 `0.0`–`1.0`。

操作成功后会将 Static Mesh 包标记为未保存，但不会自动保存。

### Analyze References

引用分析会通过 Asset Registry 查询第一个所选资产的包级引用者和依赖项。结果会排序，并根据入口显示在可停靠面板或对话框中。

报告反映 Asset Registry 当前可用的数据，并不是运行时对象引用图。

## 设置

设置页面位于 **Edit > Project Settings > Plugins > Asset Management Editor**。

| 设置 | 默认值 | 用途 |
| --- | --- | --- |
| `NamingRuleData` | `None` | 可选的项目自有规则 Data Asset，优先于内置规则执行。 |
| `BuiltInNamingRules` | 插件规则集 | 基于资产类的后备命名规则。 |
| `bConfirmDestructiveOperations` | `True` | 在重命名、整理和 LOD 操作前请求确认。 |
| `ExcludedDirectories` | 空 | 修改类操作跳过的包路径，同时包含其子路径。 |
| `bAutoComputeLODScreenSize` | `False` | 启用后由虚幻引擎计算 LOD Screen Size。 |
| `DefaultLODLevels` | 三层 | 包含 LOD0 的缩减方案。 |

首次加载时，插件默认值会复制到项目的 `Config/DefaultAssetManagementEditor.ini`。配置架构升级会按资产类补充缺失的默认命名规则，同时保留项目中已有的规则。


## 安全性与限制

- 重命名、整理和 LOD 操作会修改编辑器资产。建议使用版本控制，并在确认前检查选择内容。
- 排除目录使用包路径前缀匹配。
- 没有匹配命名规则的资产会被跳过。
- 命名和移动冲突会被报告，不会覆盖现有资产。
- 按类型整理使用原生资产类名称作为目标目录名。
- LOD 生成仅支持 Static Mesh，并且不会自动保存修改后的包。
- 引用分析只处理第一个所选资产，并报告 Asset Registry 已知的包依赖关系。
- 引用可选引擎或项目插件资产类的内置规则，仅在对应资产类可用时生效。
- 本插件仅用于编辑器，不应从运行时游戏逻辑调用。

## 仓库结构

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

## 支持

- 问题反馈：[GitHub Issues](https://github.com/givecode/AssetManagementEditor/issues)

报告构建问题时，请提供虚幻引擎版本、目标平台、完整编译错误和插件版本。

## 版权

Copyright (c) 2026 haozena. All Rights Reserved.

