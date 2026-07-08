---
name: yida-design-language
description: yida_client 项目的全局 UI 设计语言与编码约定（配色/圆角/间距/图标/透明度），美化任何页面时遵守
metadata: 
  node_type: memory
  type: project
  originSessionId: e9225572-88af-4741-b0d2-7331b4ca767e
---

yida_client（Flutter 3.38 / Dart ^3.10）的全局设计语言，美化任何页面时务必遵守：

- **配色**：主色种子 `Colors.deepPurple`，经 `ColorScheme.fromSeed` 生成。取色必须走 `Theme.of(context).colorScheme.xxx`，**严禁硬编码 `Colors.white/black/green/blue/red`**，否则暗色主题会破相。功能语义色复用 MD3 容器色板（primaryContainer / tertiaryContainer / errorContainer / onErrorContainer 等），由 fromSeed 自动派生、亮暗自适应。支持亮暗主题切换（`FreeSper.spThemeIsLight`）。
- **透明度**：统一用 `withValues(alpha: 0.x)`。项目已全量迁移，`withOpacity` 在 3.38 已 deprecated，使用会告警。
- **圆角档位**：8(chip) / 12(小项) / 16(中) / 20(卡片) / 24(大按钮)。
- **间距档位**：8 / 12 / 16 / 20 / 24。
- **MD3 surface 层级**：可用 surfaceContainer / surfaceContainerHigh / surfaceContainerHighest / surfaceContainerLowest / outlineVariant。
- **图标**：统一用 `FreeIcon`（lib/free_widget/free_icon.dart），支持 IconData(Material) / HugeIcons(List) / FaIconData；常量集中在 `FreeIconThingy`（lib/free_icon/FreeIcon.dart），如 memoryGroup / algorithm / aiMagic / cloud / fragment / reset / book / star1 / analysis 等。
- **按钮**：`FilledButton.tonalIcon` / `FilledButton.tonal` 可直接用（M3 标准）。
- **取 context**：优先用 `build(context)` 入参的 context 取 Theme，避免 `Get.context!`（坏味道）。
- **主题配置**：themeLight / themeDark 在 lib/home.dart；高级设计 token 在 lib/free_page/memory_algorithm_edit_page/theme/enhanced_theme.dart。

**Why:** 这些是跨文件探索得出的项目级约定，单看一个文件无法得知，违反会导致暗色主题破相或 lint 告警。
**How to apply:** 美化任何 widget 时，颜色走 colorScheme、透明度用 withValues、圆角间距只取档位值、图标走 FreeIconThingy。参考实装：lib/home/memory_home/tab_views/memory_group_list_page/view.dart 的 `_MemoryGroupCard`。
