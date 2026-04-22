# TEST-CASE.md — VLA-NTP 在线记事本（Singer 版）

## 1. 文档信息

| 项目 | 内容 |
|------|------|
| 应用名称 | 在线记事本（VLA-NTP） |
| 项目 | Singer |
| PRD 版本 | v1.0 |
| 测试用例版本 | v1.0 |
| 编写人 | MOSS (QA) |
| 指派人 | Pichai (PM) |
| 日期 | 2026-04-22 |
| 任务编号 | SINGER-NTP-T01 |
| Repo | https://github.com/labradorsedsota/online-notepad-singer |
| 部署地址 | https://labradorsedsota.github.io/online-notepad-singer/ |
| 品鉴者 | 廖雨亭 |

---

## 2. 测试范围

PRD 第 10 章验收汇总，共 21 个验收点 + 2 条 BVA 补充用例。

| 层级 | 数量 | 说明 |
|------|------|------|
| L1 核心功能 | 8 + 1 补充 | 阻断发布 |
| L2 重要功能 | 8 + 1 补充 | 影响体验 |
| L3 边缘功能 | 5 + 2 BVA | 可降级 + 边界值分析 |
| **合计** | **25** | |

### 功能模块覆盖

| 模块 | 功能 | 覆盖用例 |
|------|------|----------|
| F-01 | 笔记标题输入 | L1.1, L3.6 |
| F-02 | 笔记内容编辑 | L1.2, L3.5 |
| F-03 | 保存笔记 | L1.3, L2.6 |
| F-04 | 导出笔记 | L1.5, L1.6, L1.7, L2.5, L2.6, L3.7 |
| F-05 | 清空笔记 | L1.8, L2.7, L3.4 |
| F-06 | 暗色模式 | L2.1, L2.2 |
| F-07 | 字体选择 | L2.3, L2.4 |
| F-08 | 按钮提示 | L3.1 |
| F-09 | 底部链接 | L3.2, L3.3 |
| F-10 | 未保存提醒 | L2.8 |
| 跨功能 | 数据持久化 | L1.4 |

---

## 3. 测试环境

| 项目 | 值 |
|------|------|
| 浏览器 | Google Chrome（最新稳定版） |
| 操作系统 | macOS（Darwin arm64） |
| 测试工具 | mano-cua（GUI 自动化，优先） |
| 部署地址 | https://labradorsedsota.github.io/online-notepad-singer/ |
| 执行规范 | mano-cua-execution-spec v1.6 |
| 设计规范 | test-case-design-spec v1.0 |

---

## 4. Fixture 清单

### 4.1 Pre-flight 脚本

所有脚本存放于 `reports/singer-ntp/fixtures/`。

| 文件名 | 用途 | 使用用例 |
|--------|------|----------|
| `preflight-clear.js` | 清除所有 vla-ntp-* localStorage 键 | 所有 CUSTOM/EMPTY 用例 |
| `preflight-inject-note.js` | 注入标准测试笔记（标题+内容） | L1.4, L1.5, L1.6, L1.7, L1.8, L2.5, L2.7 |
| `preflight-inject-dark.js` | 注入暗色模式偏好 | L2.2 |
| `preflight-inject-font-mono.js` | 注入等宽字体偏好 | L2.4 |
| `preflight-inject-saved-state.js` | 注入已保存状态（用于未保存提醒测试） | L2.8 |
| `preflight-inject-special-title.js` | 注入含特殊字符的标题 | L3.7 |
| `preflight-inject-title-100.js` | 注入 100 字符标题 | L3.6 |
| `preflight-inject-mono-test.js` | 注入等宽对齐测试内容（ABCD.../1234.../WWWW.../iiii...） | L2.3b |

### 4.2 脚本内容

**preflight-clear.js**
```javascript
['title','content','theme','font'].forEach(k => localStorage.removeItem('vla-ntp-'+k));
```

**preflight-inject-note.js**
```javascript
localStorage.setItem('vla-ntp-title', '自动化测试标题');
localStorage.setItem('vla-ntp-content', '测试内容第一行\n测试内容第二行\n测试内容第三行');
```

**preflight-inject-dark.js**
```javascript
localStorage.setItem('vla-ntp-theme', 'dark');
```

**preflight-inject-font-mono.js**
```javascript
localStorage.setItem('vla-ntp-font', 'mono');
```

**preflight-inject-saved-state.js**
```javascript
localStorage.setItem('vla-ntp-title', '已保存标题');
localStorage.setItem('vla-ntp-content', '已保存内容');
```

**preflight-inject-special-title.js**
```javascript
localStorage.setItem('vla-ntp-title', '测试/标题*文件<名>');
localStorage.setItem('vla-ntp-content', '特殊字符标题导出测试');
```

**preflight-inject-title-100.js**
```javascript
localStorage.setItem('vla-ntp-title', 'A'.repeat(100));
localStorage.setItem('vla-ntp-content', '边界值测试内容');
```

**preflight-inject-mono-test.js**
```javascript
localStorage.setItem('vla-ntp-title', '等宽对齐测试');
localStorage.setItem('vla-ntp-content', 'ABCDEFGHIJ\n1234567890\nWWWWWWWWWW\niiiiiiiiii');
```

### 4.3 localStorage Key 对照

| Key | 说明 | 来源 |
|-----|------|------|
| `vla-ntp-title` | 笔记标题 | BR-06 |
| `vla-ntp-content` | 笔记内容 | BR-06 |
| `vla-ntp-theme` | 主题（'dark'/'light'） | BR-06 |
| `vla-ntp-font` | 字体（'default'/'mono'/'serif'/'sans'） | BR-06 |

---

## 5. 冲突扫描报告（D2.4）

### 写操作用例

| 用例 | 写入目标 |
|------|----------|
| L1.3 | vla-ntp-title, vla-ntp-content |
| L1.8 | 删除 vla-ntp-title, vla-ntp-content |
| L2.1 | vla-ntp-theme |
| L2.3 | vla-ntp-font |

### 数据依赖读操作用例

| 用例 | 依赖数据 | 隔离方式 |
|------|----------|----------|
| L1.4 | vla-ntp-title, vla-ntp-content | 策略二：Pre-flight 注入 |
| L2.2 | vla-ntp-theme | 策略二：Pre-flight 注入 |
| L2.4 | vla-ntp-font | 策略二：Pre-flight 注入 |
| L2.8 | vla-ntp-title, vla-ntp-content | 策略二：Pre-flight 注入 |

### 冲突结论

所有数据依赖型用例均采用策略二（Pre-flight 注入），每条用例执行前先清空 localStorage 再注入指定数据。按 L1 → L2 → L3 顺序执行无冲突风险。

---

## 6. 测试用例 — L1 核心功能

---

### L1.1 标题输入正常显示和输入

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L1.1-001` |
| 关联 AC | AC-01-1, AC-01-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 打开 https://labradorsedsota.github.io/online-notepad-singer/ + 窗口最大化

**任务描述：**
打开页面后，观察页面顶部是否存在标题输入框，输入框内是否显示 placeholder 文字"输入笔记标题…"。然后在标题输入框中输入文字"MOSS测试笔记标题"，观察文字是否实时显示在输入框中。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-01-1) 页面加载完成后，标题输入框显示，placeholder 为"输入笔记标题…"
2. (AC-01-2) 在标题框中输入"MOSS测试笔记标题"后，文字实时显示在输入框中

---

### L1.2 内容编辑区正常显示和输入

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L1.2-001` |
| 关联 AC | AC-02-1, AC-02-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 打开部署地址 + 窗口最大化

**任务描述：**
打开页面后，观察页面中是否存在卡片容器包裹的文本编辑区域，编辑区内是否显示 placeholder 文字"开始记录…"。然后在编辑区中输入多行文字："第一行测试内容\n第二行测试内容\n第三行测试内容\n第四行测试内容\n第五行测试内容\n第六行测试内容\n第七行测试内容\n第八行测试内容\n第九行测试内容\n第十行测试内容"（每行用回车分隔），观察文字是否实时显示，编辑区是否自动扩展高度且无内部滚动条。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-02-1) 页面加载完成后，显示卡片包裹的文本编辑区域，placeholder 为"开始记录…"
2. (AC-02-2) 在编辑区输入多行文字后，文字实时显示，编辑区自动扩展高度（不出现内部滚动条，最小高度 300px）

---

### L1.3 保存到 localStorage 成功

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L1.3-001` |
| 关联 AC | AC-03-1 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 打开部署地址 + 窗口最大化

**任务描述：**
在标题输入框中输入"保存测试标题"，在内容编辑区输入"保存测试内容第一行"。然后点击"保存"按钮（带💾图标的蓝色实心按钮），观察页面顶部是否出现绿色成功提示消息"保存成功"。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-03-1) 点击"保存"按钮后，数据保存到 localStorage，页面顶部显示绿色成功提示消息"保存成功"

---

### L1.4 刷新页面恢复标题和内容

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L1.4-001` |
| 关联 AC | AC-01-3, AC-02-3, AC-03-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要 localStorage 中存在已保存的标题和内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-note.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-note.js` 注入标题"自动化测试标题"和内容"测试内容第一行\n测试内容第二行\n测试内容第三行"
3. 打开部署地址 + 窗口最大化（页面 init 函数将从 localStorage 加载数据）

**任务描述：**
打开页面后，观察标题输入框中是否显示"自动化测试标题"，内容编辑区中是否显示"测试内容第一行"、"测试内容第二行"、"测试内容第三行"（三行文字）。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-01-3) 标题输入框恢复显示上次保存的标题内容"自动化测试标题"
2. (AC-02-3) 内容编辑区恢复显示上次保存的内容"测试内容第一行\n测试内容第二行\n测试内容第三行"
3. (AC-03-3) 标题和内容均从 localStorage 正确恢复，数据未丢失

---

### L1.5 导出 TXT 文件成功

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L1.5-001` |
| 关联 AC | AC-04-1, AC-04-2, AC-04-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要编辑区有内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-note.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-note.js` 注入标题和内容
3. 打开部署地址 + 窗口最大化

**任务描述：**
页面加载后（标题和内容已预填），点击"导出"按钮（带📤图标的蓝色描边按钮）。观察是否弹出导出确认对话框。对话框中应显示格式选择（Radio 单选按钮：TXT / Markdown / HTML），且 TXT 默认选中。保持 TXT 选中状态，点击"确认导出"按钮，观察是否触发 .txt 文件下载。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-04-1) 点击"导出"按钮后，弹出导出确认对话框
2. (AC-04-2) 对话框中显示格式选择（Radio：TXT / Markdown / HTML），默认选中 TXT
3. (AC-04-3) 选择 TXT 格式并点击"确认导出"后，下载 .txt 文件，文件名为标题内容（无标题则为"未命名笔记"），文件内容为纯文本

---

### L1.6 导出 Markdown 文件成功

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L1.6-001` |
| 关联 AC | AC-04-4 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要编辑区有内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-note.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-note.js` 注入标题和内容
3. 打开部署地址 + 窗口最大化

**任务描述：**
页面加载后（标题和内容已预填），点击"导出"按钮。在导出对话框中，选择 Markdown 格式（点击"Markdown"对应的 Radio 单选按钮），然后点击"确认导出"按钮，观察是否触发 .md 文件下载。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-04-4) 选择 Markdown 格式并点击"确认导出"后，下载 .md 文件，标题作为一级标题（`# 标题`），正文在标题下方

---

### L1.7 导出 HTML 文件成功

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L1.7-001` |
| 关联 AC | AC-04-5 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要编辑区有内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-note.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-note.js` 注入标题和内容
3. 打开部署地址 + 窗口最大化

**任务描述：**
页面加载后（标题和内容已预填），点击"导出"按钮。在导出对话框中，选择 HTML 格式（点击"HTML"对应的 Radio 单选按钮），然后点击"确认导出"按钮，观察是否触发 .html 文件下载。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-04-5) 选择 HTML 格式并点击"确认导出"后，下载 .html 文件，含完整 HTML 结构（`<html><head><title>标题</title></head><body>...`），正文段落用 `<p>` 包裹（按换行分段）

---

### L1.8 清空功能正常

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L1.8-001` |
| 关联 AC | AC-05-1, AC-05-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要编辑区有内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-note.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-note.js` 注入标题和内容
3. 打开部署地址 + 窗口最大化

**任务描述：**
页面加载后（标题和内容已预填），点击"清空"按钮（带🗑图标的红色描边按钮）。观察是否出现确认提示"确定要清空所有内容吗？此操作不可撤销"。点击确认（OK），观察标题输入框和内容编辑区是否被清空。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-05-1) 点击"清空"按钮后，显示确认提示"确定要清空所有内容吗？此操作不可撤销"
2. (AC-05-2) 用户确认清空后，标题和内容清空，localStorage 中的保存数据也清除

---

### L1.4b [补充] 端到端保存-恢复（品鉴者要求）

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L1.4b-001` |
| 关联 AC | AC-01-3, AC-02-3, AC-03-1, AC-03-3 |
| 设计技术 | PRD 追溯（端到端路径补充） |
| 数据依赖 | EMPTY |
| 冲突标记 | ⚠ 与 L1.3 写入同 key，但本用例自含完整保存流程，Pre-flight 已清空 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 打开部署地址 + 窗口最大化

**任务描述：**
打开空白页面后，在标题输入框中输入"端到端恢复测试标题"，在内容编辑区输入"端到端恢复测试内容第一行"并回车输入"第二行内容"。然后点击"保存"按钮（带💾图标的蓝色实心按钮），等待出现"保存成功"提示。提示消失后，刷新页面（按 F5 或 Cmd+R）。页面重新加载后，观察标题输入框是否仍显示"端到端恢复测试标题"，内容编辑区是否仍显示"端到端恢复测试内容第一行"和"第二行内容"。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-03-1) 点击"保存"按钮后，显示绿色成功提示消息"保存成功"
2. (AC-01-3) 刷新页面后，标题输入框仍显示原始输入的标题"端到端恢复测试标题"
3. (AC-02-3) 刷新页面后，内容编辑区仍显示原始输入的内容"端到端恢复测试内容第一行"和"第二行内容"
4. (AC-03-3) 标题和内容在刷新后完整恢复，数据未丢失

**与 L1.4 的区别：** L1.4 通过 Pre-flight 注入 localStorage 验证恢复逻辑；本用例走完整用户操作链路（输入→保存→刷新→验证），覆盖真实保存-恢复端到端路径。

---

## 7. 测试用例 — L2 重要功能

---

### L2.1 暗色模式切换正常

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L2.1-001` |
| 关联 AC | AC-06-1, AC-06-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage（确保默认亮色模式）
2. 打开部署地址 + 窗口最大化

**任务描述：**
打开页面后，确认当前为亮色模式（背景为浅灰白色）。找到页面右上方工具栏中"暗色"标签旁的 Switch 开关，点击 Switch 使其变为开启状态，观察页面是否切换为暗色模式（背景变为深色、文字变为浅色）。然后再次点击 Switch 使其变为关闭状态，观察页面是否切换回亮色模式。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-06-1) 切换 Switch 为开启状态后，页面切换为暗色模式（背景深色、文字浅色）
2. (AC-06-2) 切换 Switch 为关闭状态后，页面切换回亮色模式

---

### L2.2 暗色模式状态持久化

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L2.2-001` |
| 关联 AC | AC-06-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要 localStorage 中存在暗色模式偏好 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-dark.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-dark.js` 注入 `vla-ntp-theme=dark`
3. 打开部署地址 + 窗口最大化

**任务描述：**
打开页面后，观察页面是否以暗色模式加载（背景为深色 #1C1C1E，文字为浅色），并且工具栏中"暗色"旁的 Switch 开关是否处于开启状态。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-06-3) 页面刷新后，保持上次选择的暗色模式（背景深色、文字浅色），Switch 处于开启状态

---

### L2.3 字体选择切换正常

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L2.3-001` |
| 关联 AC | AC-07-1, AC-07-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要编辑区有内容以便观察字体变化 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-note.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-note.js` 注入内容（便于观察字体变化）
3. 打开部署地址 + 窗口最大化

**任务描述：**
页面加载后，找到工具栏中"字体"标签旁的下拉菜单。点击下拉菜单，观察是否至少包含以下选项：默认、等宽（Monospace）、衬线（Serif）、无衬线（Sans-serif）。依次选择"等宽"，观察内容编辑区文字是否立即切换为等宽字体；再选择"衬线"，观察是否切换为衬线字体；再选择"无衬线"，观察是否切换为无衬线字体；最后选回"默认"。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-07-1) 字体下拉菜单显示，至少包含：默认、等宽（Monospace）、衬线（Serif）、无衬线（Sans-serif）
2. (AC-07-2) 选择一种字体后，编辑区文字立即切换为对应字体

---

### L2.3b [补充] 等宽字体特征验证（品鉴者要求）

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L2.3b-001` |
| 关联 AC | AC-07-2（强化） |
| 设计技术 | PRD 追溯（字体特征深度验证） |
| 数据依赖 | CUSTOM — 需要编辑区有含字母和数字的内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-mono-test.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-mono-test.js` 注入等宽对齐测试内容
3. 打开部署地址 + 窗口最大化

**任务描述：**
页面加载后，编辑区已预填如下内容（用于验证等宽对齐）：
```
ABCDEFGHIJ
1234567890
WWWWWWWWWW
iiiiiiiiii
```
找到工具栏中"字体"标签旁的下拉菜单，选择"等宽"。观察编辑区中上述四行文字是否每行等宽对齐——即每行的字符宽度一致，"W"和"i"所占的水平宽度相同，四行文字的右边缘应大致对齐。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-07-2 强化) 选择"等宽"字体后，编辑区文字切换为等宽字体
2. (AC-07-2 强化) 字母"W"和字母"i"所占水平宽度相同，四行文字（ABCDEFGHIJ / 1234567890 / WWWWWWWWWW / iiiiiiiiii）的右边缘大致对齐，确认等宽特征生效

**与 L2.3 的区别：** L2.3 验证字体切换是否生效（有无变化）；本用例验证等宽字体的实际视觉特征（字母/数字是否等宽对齐）。

---

### L2.4 字体选择状态持久化

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L2.4-001` |
| 关联 AC | AC-07-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要 localStorage 中存在字体偏好 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-font-mono.js` + `preflight-inject-note.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-font-mono.js` 注入 `vla-ntp-font=mono`
3. 执行 `preflight-inject-note.js` 注入内容（便于观察字体）
4. 打开部署地址 + 窗口最大化

**任务描述：**
打开页面后，观察工具栏中的字体下拉菜单是否显示"等宽"为当前选中项，并且内容编辑区的文字是否以等宽字体（Monospace）显示。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-07-3) 页面刷新后，保持上次选择的字体（下拉菜单显示"等宽"，编辑区文字为等宽字体）

---

### L2.5 导出对话框取消关闭

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L2.5-001` |
| 关联 AC | AC-04-6 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要编辑区有内容（否则会被空内容拦截） |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-note.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-note.js` 注入内容
3. 打开部署地址 + 窗口最大化

**任务描述：**
页面加载后（内容已预填），点击"导出"按钮打开导出对话框。对话框打开后，点击"取消"按钮，观察对话框是否关闭且未执行导出操作。然后再次点击"导出"按钮打开对话框，这次点击对话框外部的遮罩区域，观察对话框是否关闭且未执行导出。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-04-6) 点击"取消"后，对话框关闭，不执行导出
2. (AC-04-6) 点击对话框外部后，对话框关闭，不执行导出

---

### L2.6 空内容阻止保存和导出

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L2.6-001` |
| 关联 AC | AC-03-2, AC-04-7 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 打开部署地址 + 窗口最大化

**任务描述：**
打开页面后，不输入任何内容（标题和内容均为空）。先点击"保存"按钮，观察是否出现警告提示"内容为空，无法保存"。等待提示消失后，点击"导出"按钮，观察是否出现警告提示"内容为空，无法导出"，且不弹出导出对话框。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-03-2) 标题和内容均为空时，点击"保存"按钮，显示警告提示"内容为空，无法保存"
2. (AC-04-7) 标题和内容均为空时，点击"导出"按钮，显示警告提示"内容为空，无法导出"，不弹出对话框

---

### L2.7 清空确认和取消

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L2.7-001` |
| 关联 AC | AC-05-1, AC-05-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要编辑区有内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-note.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-note.js` 注入标题和内容
3. 打开部署地址 + 窗口最大化

**任务描述：**
页面加载后（标题和内容已预填），点击"清空"按钮。观察是否出现确认提示"确定要清空所有内容吗？此操作不可撤销"。点击取消（Cancel），观察标题和内容是否保持不变。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-05-1) 点击"清空"按钮后，显示确认提示"确定要清空所有内容吗？此操作不可撤销"
2. (AC-05-3) 用户取消后，标题和内容不做任何改变

---

### L2.8 未保存提醒显示和消失

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L2.8-001` |
| 关联 AC | AC-10-1, AC-10-2, AC-10-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要已保存的基线状态 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-saved-state.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-saved-state.js` 注入已保存状态（标题"已保存标题"、内容"已保存内容"）
3. 打开部署地址 + 窗口最大化

**任务描述：**
页面加载后（已保存的标题和内容已恢复），首先确认页面顶部没有显示"您有未保存的更改"的黄色 Alert 提醒条。然后在内容编辑区中追加输入"新增未保存文字"，观察页面顶部是否出现黄色 Alert 提醒条"您有未保存的更改"。接着点击"保存"按钮，观察保存成功后 Alert 提醒条是否消失。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-10-3) 页面加载后，内容未修改（与上次保存一致），不显示 Alert
2. (AC-10-1) 修改内容后（追加"新增未保存文字"），页面顶部显示 Alert 提醒"您有未保存的更改"
3. (AC-10-2) 点击保存后，Alert 消失

---

## 8. 测试用例 — L3 边缘功能

---

### L3.1 按钮 Tooltip 显示和消失

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L3.1-001` |
| 关联 AC | AC-08-1, AC-08-2, AC-08-3, AC-08-4 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | ANY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 打开部署地址 + 窗口最大化

**任务描述：**
找到页面中的三个操作按钮：保存（💾）、导出（📤）、清空（🗑）。将鼠标悬停在"保存"按钮上方并等待至少 1 秒，观察是否出现 Tooltip 提示"保存笔记到浏览器"。然后将鼠标移出按钮区域，观察 Tooltip 是否消失。对"导出"按钮重复同样操作，观察 Tooltip 是否显示"导出为文件"。对"清空"按钮重复同样操作，观察 Tooltip 是否显示"清空所有内容"。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-08-1) 鼠标悬停"保存"按钮超过 300ms 后，显示 Tooltip "保存笔记到浏览器"
2. (AC-08-2) 鼠标悬停"导出"按钮超过 300ms 后，显示 Tooltip "导出为文件"
3. (AC-08-3) 鼠标悬停"清空"按钮超过 300ms 后，显示 Tooltip "清空所有内容"
4. (AC-08-4) 鼠标移出按钮区域后，Tooltip 消失

---

### L3.2 底部关于链接可点击

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L3.2-001` |
| 关联 AC | AC-09-1, AC-09-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | ANY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 打开部署地址 + 窗口最大化

**任务描述：**
滚动到页面底部，观察是否存在"关于"和"帮助"两个链接。点击"关于"链接，观察是否弹出关于信息对话框，对话框中应包含应用名称、版本信息。如弹出对话框则关闭。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-09-1) 页面底部显示"关于"和"帮助"两个链接
2. (AC-09-2) 点击"关于"链接后，显示关于信息（对话框或跳转到 repo 页面）

---

### L3.3 底部帮助链接可点击

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L3.3-001` |
| 关联 AC | AC-09-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | ANY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 打开部署地址 + 窗口最大化

**任务描述：**
滚动到页面底部，点击"帮助"链接，观察是否显示使用说明信息。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-09-3) 点击"帮助"链接后，显示使用说明信息

---

### L3.4 空白状态点清空提示

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L3.4-001` |
| 关联 AC | AC-05-4 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 打开部署地址 + 窗口最大化

**任务描述：**
打开页面后，不输入任何内容（标题和内容均为空），直接点击"清空"按钮。观察是否出现信息提示"已经是空白状态"，而非确认对话框。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-05-4) 标题和内容均为空时，点击"清空"按钮，显示信息提示"已经是空白状态"

---

### L3.5 编辑区 placeholder 显示

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L3.5-001` |
| 关联 AC | AC-02-4 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 打开部署地址 + 窗口最大化

**任务描述：**
打开页面后，不输入任何内容，观察内容编辑区是否显示 placeholder 文字"开始记录…"（灰色次要文字）。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (AC-02-4) 编辑区为空时，显示 placeholder 文字"开始记录…"

---

## 9. 测试用例 — BVA 补充（D1）

---

### L3.6 [BVA] 标题最大长度边界

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L3.6-001` |
| 关联 AC | AC-01-2（衍生） |
| 设计技术 | BVA — 标题字段上界（BR-01: 最多 100 字符） |
| 目标边界 | 标题长度 = 100（有效最大值） |
| 数据依赖 | CUSTOM — 注入 100 字符标题 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-title-100.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-title-100.js` 注入 100 字符标题（"A" x 100）
3. 打开部署地址 + 窗口最大化

**任务描述：**
打开页面后，观察标题输入框是否完整显示 100 个字符。然后点击标题输入框末尾，尝试输入一个额外字符"X"，观察标题是否仍然保持 100 字符（额外字符被拒绝，因为 HTML maxlength="100"）。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (BR-01) 标题输入框正确显示 100 个字符
2. (BR-01) 尝试输入第 101 个字符时被拒绝，标题保持 100 字符（截断）

---

### L3.7 [BVA] 导出文件名特殊字符处理

| 项 | 值 |
|---|---|
| mosstid | `singer-ntp-v1.0-L3.7-001` |
| 关联 AC | AC-04-3（衍生） |
| 设计技术 | BVA — 文件名特殊字符（BR-04: `/\:*?"<>\|` 替换为 `_`） |
| 目标边界 | 标题含 BR-04 定义的特殊字符 |
| 数据依赖 | CUSTOM — 注入含特殊字符的标题 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `preflight-clear.js` + `preflight-inject-special-title.js` |

**Pre-flight：**
1. 执行 `preflight-clear.js` 清空 localStorage
2. 执行 `preflight-inject-special-title.js` 注入标题"测试/标题*文件<名>"和内容
3. 打开部署地址 + 窗口最大化

**任务描述：**
页面加载后（标题已预填为"测试/标题*文件<名>"），点击"导出"按钮，保持 TXT 格式选中，点击"确认导出"，观察是否成功触发文件下载。观察下载的文件名中特殊字符 `/`、`*`、`<`、`>` 是否被替换为 `_`。仅在当前页面操作，不要导航到其他网址。

**Expected Results（逐条）：**
1. (BR-04) 导出成功触发文件下载
2. (BR-04) 文件名中的 `/\:*?"<>|` 字符被替换为 `_`

---

## 10. 执行注意事项

### 10.1 通用 Pre-flight 流程

每条用例执行前，按以下顺序执行 Pre-flight（对应 mano-cua-execution-spec 条款 1c → 1 → 1a → 1b → 2）：

1. **数据隔离**（条款 1c）：按用例声明的数据依赖执行清理
2. **页面重置 + 最大化**（条款 1）：`open -a "Google Chrome" <URL>` + 窗口最大化脚本
3. **策略判定**（条款 1a）：已在每条用例中标注
4. **Pre-flight 注入**（条款 1b）：策略二用例执行 JS 注入 + 刷新页面
5. **URL 确认**（条款 2）：确认 Chrome 当前标签页 URL 为部署地址

### 10.2 关键条款速查

| 条款 | 要点 |
|------|------|
| 条款 3 | 任务指令对齐 PRD 原文关键词 |
| 条款 5 | 日志 tee 到本地 |
| 条款 6/7 | 首步 URL 校验 + 自主导航拦截 |
| 条款 6a/7a | 重试上限 3 次，BLOCKED 必须附诊断 |
| 条款 7b | 软超时 600s / 硬超时 900s |
| 条款 9 | expected result 逐条列出（已在每条用例中完成） |
| 条款 9a | 计数匹配 + 关键词命中 |
| 条款 11 | evaluation 与日志交叉验证 |
| 条款 14 | 关闭测试标签页（MOSS 操作，contains 匹配） |

### 10.3 matchPath 配置

关闭标签页时使用的 matchPath：`online-notepad-singer`

### 10.4 日志路径模板

```
reports/singer-ntp/logs/2026-04-XX/singer-ntp-<测试点ID>.log
```

---

## 文档合规 Checklist

- [x] 文档信息
- [x] 测试范围
- [x] 测试用例 L1（8 项 + 1 补充）
- [x] 测试用例 L2（8 项 + 1 补充）
- [x] 测试用例 L3（5 项 + 2 BVA）
- [x] Fixture 清单（7 个 Pre-flight 脚本 + 内容）
- [x] 测试环境
- [x] 冲突扫描报告（D2.4）
- [x] BVA 边界值分析（D1）
- [x] 数据隔离声明（D2）
- [x] mosstid 格式：`singer-ntp-v1.0-{测试点}-{序号}`
