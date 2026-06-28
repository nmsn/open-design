# 教训：教学前必须验证

在第 8 课中使用了虚构的目录名 `skills/deck-creator/SKILL.md`，但实际上 Open Design 中没有这个目录。真实的 deck 类技能叫 `frontend-slides`、`deck-open-slide-canvas` 等。

**Why:** 占位符/示例代码会让学员去 `ls` 找不到，浪费学习时间并降低对课程的信任。

**How to apply:**
- 课程中所有具体路径、文件名、目录名必须先 `ls` 验证
- 复制真实代码片段而非自造
- 不确定的示例明确标注 "示例路径，以项目实际为准"
- 复杂结构用 `cat <file>` 复制前 20-30 行，比手写更准