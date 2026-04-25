# TwSkill

Claude Code 自定义技能集合。

## 技能列表

| 技能 | 说明 |
|------|------|
| [rnoh-lib-guide](./rnoh-lib-guide/) | 鸿蒙 React Native 第三方库接入指南 |

## 使用方式

将技能目录复制到 `~/.claude/skills/` 下即可在 Claude Code 中使用。

```bash
# 示例：安装 rnoh-lib-guide
cp -r rnoh-lib-guide ~/.claude/skills/
cd ~/.claude/skills/rnoh-lib-guide/
git clone https://gitee.com/react-native-oh-library/usage-docs.git references/usage-docs
```
