# 鸿蒙 React Native 第三方库使用指南

## 任务目标
帮调用理解一个鸿蒙 RN 第三方库：
- 它是什么、有哪些版本
- 怎么安装（JS 侧 npm、原生侧 ohpm）
- 需要什么配置（ManualLink、权限、工程级配置）
- 有什么限制和注意事项（API 版本约束、遗留问题）

## 前置准备
- **RN 版本**：默认使用 0.77。如果用户没有明确指定版本，直接使用 0.77，并在回复中说明"基于 RN 0.77 查询"
- **文档根目录**：references/usage-docs/zh-cn/
- **首次使用**：需将 usage-docs 克隆到技能目录下：
  ```bash
  cd ~/.claude/skills/rnoh-lib-guide/
  git clone https://gitee.com/react-native-oh-library/usage-docs.git references/usage-docs
  ```
- **版本对应总表**：参见 references/usage-docs/zh-cn/README.md

## 操作步骤

### 第一步：定位库文档

1. **识别原始库名**
   - 从用户请求中提取原始库名
   - 处理不同格式：
     - scoped 包：@react-native-async-storage/async-storage → 关键字：async-storage
     - unscoped 包：react-native-svg → 关键字：svg
     - 带连字符的：react-native-clipboard-clipboard → 关键字：clipboard

2. **渐进式定位库文档**
   - **步骤1：精确搜索**：使用 `find references/usage-docs/zh-cn -name "*{关键字}*.md"`
   - **步骤2：模糊搜索**（如果步骤1结果>1）：使用更宽松的关键字
   - **步骤3：README 搜索**（如果步骤2仍无结果）：在 README 中搜索原始库名
   - **步骤4：读取文档**：根据最终确定的文件路径读取

### 第二步：提取关键信息

通读文档全文，提取以下信息（根据文档内容自由判断，不限定字段）：

- **版本映射表**：提取所有可用版本和对应的 RN 版本
- **安装方式**：识别 JS 侧和原生侧的安装方式
  - JS 侧：通常用 npm/yarn
  - 原生侧：检查是否需要 ohpm install（通过 oh-package.json5 配置判断）
- **Autolink 支持**：判断是否支持 Autolink，如不支持需要 ManualLink
- **ManualLink 配置**：⚠️ 见第三步的验证要求
- **依赖库**：提取文档中声明的依赖（如"本库依赖 react-native-reanimated"），包括：
  - 依赖库名和鸿蒙包名
  - 依赖的版本要求（如"reanimated 安装 v3.6.2 配套使用"）
  - 是否为可选依赖（"如已引入则无需再次引入"）
- **Import 别名**：检查是否有"import 库名不变"的提示
- **工程级配置**：提取涉及 AppScope、configuration.json 的配置
- **权限配置**：提取需要的权限，特别标注 system_basic 等级的 ACL 签名要求
- **特定 API 约束**：提取特定接口的 API 版本要求（如 getFontScale 需要 API 18+）
- **兼容性信息**：提取 RNOH、SDK、IDE、ROM 版本要求
- **遗留问题**：提取标注为 "no" 或 "partially" 的属性/接口

### 第三步：验证 ManualLink 配置（关键步骤）

⚠️ 这一步是防止编译报错的核心。不同库的 ManualLink 配置层级不同，**必须严格按文档步骤操作，不能想当然地给所有库加同样的配置。**

**验证规则：**

1. **逐条对照文档**：文档中提到的每一个需要修改的文件，都必须在报告中列出。文档中没有提到的文件，不能自己添加。
   - 有些库只需要 CMakeLists + PackageProvider（C++ 侧），不需要 RNPackagesFactory.ets（ArkTS 侧）
   - 有些库需要完整的 CMakeLists + PackageProvider + RNPackagesFactory.ets
   - 判断依据：**文档中是否有 RNPackagesFactory 的配置步骤**，有则加，没有则不加

2. **oh-package.json5 依赖 key 命名**：key 名取决于 har 包内部的 oh-package.json5 的 name 字段，可能与 npm 包名不同（如 `pull-to-refresh` vs `pull_to_refresh`）。必须从文档中复制 key 名，不能直接用 npm 包名。

3. **配置文件清单**：在报告中明确列出每个需要修改的文件，标注"需要/不需要"：
   - CMakeLists.txt：需要/不需要
   - PackageProvider.cpp：需要/不需要
   - RNPackagesFactory.ets：需要/不需要
   - entry/oh-package.json5：需要/不需要
   - harmony/oh-package.json5（overrides）：需要/不需要
   - module.json5（权限）：需要/不需要
   - 其他工程级配置：需要/不需要

### 第四步：依赖兼容性检查

必须执行，防止版本不兼容导致运行时崩溃。

1. **RNOH 版本要求**：从文档中提取要求的具体 RNOH 版本（如 0.77.86）
2. **项目实际版本**：读取项目 package.json 中的 @react-native-oh/react-native-harmony 版本
3. **版本差距风险**：如果文档要求版本 > 项目版本，必须在反馈中标注风险
4. **内部 API 依赖**：检查文档是否提到对 RNOH 内部组件的依赖（如 RNScrollView、RNImageView 等）
5. **peerDependencies 检查**：如果文档提供了库的 package.json 信息，检查 peerDependencies 中是否有特殊依赖

### 第五步：输出固定格式报告

用自然语言总结给用户，**必须包含以下固定段落**：

#### 段落1：版本信息
- 这是什么库、鸿蒙包名是什么
- 可用版本列表（版本号 + 对应 RN 版本 + Autolink 支持）
- 推荐版本

#### 段落2：安装步骤
- JS 侧安装命令（含具体版本号）
- 原生侧安装步骤（如需 ohpm）
- 如果 npm 上只有 -rc 预发布版本，需提醒

#### 段落3：ManualLink 配置（如需）
- 按文件逐条列出，每个文件标注"需要/不需要"
- 每个需要修改的文件给出具体的代码片段
- ⚠️ 明确标注哪些文件不需要修改（避免误加）

#### 段落4：依赖库（如需）
- 列出文档中声明的依赖库
- 每个依赖标注：鸿蒙包名、版本要求、是否已安装
- 如果项目已安装该依赖，标注"已安装，无需重复引入"
- 如果项目未安装，标注"需要先安装"并给出安装指引

#### 段落5：依赖兼容性检查结果
- 文档要求的 RNOH 版本 vs 项目实际版本
- 是否存在内部 API 依赖风险
- 版本差距风险提示

#### 段落6：注意事项
- Import 路径是否不变
- 不支持/部分支持的属性和接口
- 权限配置要求
- 其他兼容性信息

## 错误处理

- **库文档未找到**：尝试更宽松的关键字搜索；仍无结果则提示用户该库可能未收录
- **版本对应表无匹配版本**：选择最接近的版本，或提示用户该库可能不支持当前 RN 版本
- **文档格式异常**：如果无法解析表格，提取文档中的关键说明文字
