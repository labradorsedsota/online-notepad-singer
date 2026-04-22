# TEST-REPORT.md — 在线记事本 (Singer) 测试报告

| 项 | 值 |
|---|---|
| 产品 | 在线记事本 v1.0 (Singer) |
| 部署地址 | https://labradorsedsota.github.io/online-notepad-singer/ |
| Repo | https://github.com/labradorsedsota/online-notepad-singer |
| TEST-CASE 版本 | commit `f4a22b5` (25 条用例) |
| 测试执行者 | MOSS |
| PM | Pichai (lynx_bot) |
| 品鉴者 | 廖雨亭 |
| 执行日期 | 2026-04-22 |
| 执行方式 | mano-cua (L1.1–L1.4) + AppleScript/JS + 源码审计 (L1.4b–L3.7) |

---

## 1. 执行摘要

| 指标 | 数值 |
|---|---|
| 用例总数 | 25 |
| PASS | 14 |
| FAIL | 8 |
| CONDITIONAL PASS | 3 |
| 缺陷总数 | 5 |
| Critical | 1 |
| Major | 3 |
| Minor | 1 |

---

## 2. 缺陷汇总

### BUG-001 [Critical] Save 函数 localStorage Key 互换

| 项 | 值 |
|---|---|
| 关联用例 | L1.4, L1.4b |
| 关联 AC | AC-03-1, AC-03-3 |
| 严重程度 | Critical |

**现象：** 保存后刷新页面，标题和内容互换位置。

**根因：** `btnSave` 事件处理器中 localStorage 写入顺序错误：
```javascript
// 第 161-162 行
localStorage.setItem(KEY_TITLE, content);   // BUG: 应为 title
localStorage.setItem(KEY_CONTENT, title);   // BUG: 应为 content
```

**复现：** 输入标题 "A"、内容 "B" → 保存 → 刷新 → 标题显示 "B"、内容显示 "A"。

**验证数据：**
```
保存前 DOM:  titleInput="E2E-Save", contentArea="E2E-Line1 E2E-Line2"
localStorage: vla-ntp-title="E2E-Line1 E2E-Line2", vla-ntp-content="E2E-Save"  ← 互换
刷新后 DOM:  titleInput="E2E-Line1 E2E-Line2", contentArea="E2E-Save"  ← 互换
```

---

### BUG-002 [Major] 标题输入框文字与背景同色，不可见

| 项 | 值 |
|---|---|
| 关联用例 | L1.1 |
| 关联 AC | AC-01-2 |
| 严重程度 | Major |

**现象：** 亮色模式下，标题输入框中的文字完全不可见。

**根因：** CSS `.title-input` 设置 `color: var(--bg)` — 文字颜色等于页面背景色：
- 亮色模式：`--bg: #FAFAFA`，文字 #FAFAFA 在白色底上不可见
- 暗色模式：`--bg: #1C1C1E`，文字 #1C1C1E 在深色底上不可见

**修复建议：** 将 `color: var(--bg)` 改为 `color: var(--text)`。

---

### BUG-003 [Major] 暗色模式切换无视觉效果

| 项 | 值 |
|---|---|
| 关联用例 | L2.1 |
| 关联 AC | AC-06-1, AC-06-2 |
| 严重程度 | Major |

**现象：** 点击暗色模式 Switch，页面无任何视觉变化。

**根因：** 暗色模式事件处理器中，实际切换主题的代码被注释掉了：
```javascript
// 第 143-144 行
darkToggle.addEventListener('change', function() {
  if (this.checked) {
    // document.documentElement.setAttribute('data-theme', 'dark');   ← 注释掉了
    localStorage.setItem(KEY_THEME, 'dark');
  } else {
    // document.documentElement.removeAttribute('data-theme');        ← 注释掉了
    localStorage.setItem(KEY_THEME, 'light');
  }
});
```

**注意：** init 函数中的恢复逻辑未被注释，因此刷新后暗色模式可正常恢复。仅切换操作的即时视觉反馈失效。

---

### BUG-004 [Major] 字体选择映射全部错误

| 项 | 值 |
|---|---|
| 关联用例 | L2.3, L2.3b |
| 关联 AC | AC-07-1, AC-07-2 |
| 严重程度 | Major |

**现象：** 选择"等宽"字体后，实际应用的是衬线字体；选择"衬线"变无衬线；选择"无衬线"变等宽。

**根因：** `setFont()` 函数中 CSS 变量映射全部错乱：
```javascript
function setFont(key) {
  const map = {
    default: 'var(--font-default)',
    mono: 'var(--font-serif)',     // BUG: 等宽 → 应为 --font-mono
    serif: 'var(--font-sans)',     // BUG: 衬线 → 应为 --font-serif
    sans: 'var(--font-mono)'      // BUG: 无衬线 → 应为 --font-sans
  };
}
```

**映射关系：**

| 用户选择 | 期望字体 | 实际字体 |
|---------|---------|---------|
| 等宽 (mono) | `--font-mono` | `--font-serif` (衬线) |
| 衬线 (serif) | `--font-serif` | `--font-sans` (无衬线) |
| 无衬线 (sans) | `--font-sans` | `--font-mono` (等宽) |

---

### BUG-005 [Minor] sanitizeFilename 过滤非 ASCII 字符

| 项 | 值 |
|---|---|
| 关联用例 | L3.7 |
| 关联 AC | AC-04-3 |
| 严重程度 | Minor |

**现象：** 中文标题导出时文件名变为 "未命名笔记"。

**根因：** `sanitizeFilename()` 正则 `/[a-zA-Z0-9_\-]/g` 仅保留 ASCII 字母数字 + 下划线 + 连字符，所有中文字符被过滤，结果为空时回退到 "未命名笔记"。

---

## 3. 逐条测试结果

### L1 — 核心功能

| ID | 用例名 | AC | 判定 | 方式 | 说明 |
|---|---|---|---|---|---|
| L1.1 | 标题输入框可用 | AC-01-1, AC-01-2 | **FAIL** | mano-cua | AC-01-1(PASS:焦点可获取) AC-01-2(FAIL:文字不可见,BUG-002) |
| L1.2 | 正文编辑区可用 | AC-02-1, AC-02-2 | **PASS** | mano-cua | 输入正常、placeholder 显示正确、自动扩展正常 |
| L1.3 | 未保存提醒出现 | AC-01-2, AC-10-1 | **PASS** | mano-cua | 输入内容后黄色警告条"您有未保存的更改"正确显示 |
| L1.4 | 保存并从 localStorage 恢复 | AC-01-3, AC-02-3, AC-03-1, AC-03-3 | **FAIL** | mano-cua | 内容恢复正常(PASS) 但标题和内容值互换(BUG-001) |
| L1.4b | 端到端保存-恢复 | AC-01-3, AC-02-3, AC-03-1, AC-03-3 | **FAIL** | AppleScript+JS | 保存后localStorage中title/content互换；刷新后DOM值互换(BUG-001) |
| L1.5 | 导出 TXT 文件 | AC-04-1, AC-04-2, AC-04-3 | **PASS** | AppleScript+JS | 对话框3格式/TXT默认(PASS)；下载ExportTitle.txt,内容"ExportTitle\n\nExportContent"(PASS) |
| L1.6 | 导出 Markdown 文件 | AC-04-4 | **PASS** | AppleScript+JS | 下载ExportTitle.md,内容"# ExportTitle\n\nExportContent"(PASS) |
| L1.7 | 导出 HTML 文件 | AC-04-5 | **PASS** | AppleScript+JS | 下载ExportTitle.html,完整HTML结构含\<h1\>标题(PASS) |
| L1.8 | 清空功能 | AC-05-1, AC-05-2 | **PASS** | 源码审计 | confirm('确定要清空所有内容吗？此操作不可撤销')文案匹配AC-05-1；确认后清空DOM+localStorage(AC-05-2) |

### L2 — 重要功能

| ID | 用例名 | AC | 判定 | 方式 | 说明 |
|---|---|---|---|---|---|
| L2.1 | 暗色模式切换 | AC-06-1, AC-06-2 | **FAIL** | 源码审计 | setAttribute代码被注释,切换无视觉效果(BUG-003) |
| L2.2 | 暗色模式持久化 | AC-06-3 | **COND. PASS** | 源码审计 | localStorage正确保存'dark';init正确恢复setAttribute;但依赖L2.1先触发(L2.1 FAIL) |
| L2.3 | 字体选择切换 | AC-07-1, AC-07-2 | **FAIL** | 源码审计 | 字体映射全部错乱(BUG-004),选等宽得衬线,选衬线得无衬线,选无衬线得等宽 |
| L2.3b | 等宽字体特征验证 | AC-07-1 | **FAIL** | 源码审计 | 选择"等宽"后实际应用--font-serif,W和i宽度不等(BUG-004) |
| L2.4 | 字体选择持久化 | AC-07-3 | **COND. PASS** | 源码审计 | localStorage正确保存font值;init正确恢复;但恢复的字体仍是错误映射(BUG-004) |
| L2.5 | 导出对话框取消 | AC-04-1 | **PASS** | 源码审计 | exportCancel关闭对话框,点击遮罩也关闭,内容不丢失 |
| L2.6 | 空内容阻止保存和导出 | AC-03-2, AC-04-1 | **PASS** | 源码审计 | 空内容点保存→"内容为空，无法保存";空内容点导出→"内容为空，无法导出" |
| L2.7 | 清空确认和取消 | AC-05-1, AC-05-2 | **PASS** | 源码审计 | confirm()取消→无操作;确认→清空DOM+localStorage+显示"已清空" |
| L2.8 | 未保存提醒 | AC-10-1 | **PASS** | 源码审计+mano-cua | checkUnsaved()比较当前值与savedTitle/savedContent,变化时显示警告条 |

### L3 — 边缘/次要

| ID | 用例名 | AC | 判定 | 方式 | 说明 |
|---|---|---|---|---|---|
| L3.1 | 按钮 Tooltip | AC-08-1 | **PASS** | 源码审计 | 3个button均包裹在.tooltip-wrapper中,hover 300ms后显示tooltip |
| L3.2 | 底部"关于"链接 | AC-09-1 | **PASS** | 源码审计 | linkAbout打开aboutDialog,显示"在线记事本 v1.0"版本信息 |
| L3.3 | 底部"帮助"链接 | AC-09-2 | **PASS** | 源码审计 | linkHelp打开helpDialog,显示使用说明 |
| L3.4 | 空白状态清空提示 | AC-05-1 | **PASS** | 源码审计 | 空白时点清空→showMessage('已经是空白状态','info') |
| L3.5 | 编辑区 placeholder | AC-02-1 | **PASS** | 源码审计 | title: "输入笔记标题…", content: "开始记录…" |
| L3.6 | 标题最大长度 BVA | AC-01-1 | **COND. PASS** | 源码审计 | maxlength="100"浏览器原生限制,但文字不可见无法人工确认(BUG-002) |
| L3.7 | 导出文件名特殊字符 | AC-04-3 | **FAIL** | 源码审计 | sanitizeFilename仅保留ASCII字母数字,中文标题→"未命名笔记"(BUG-005) |

---

## 4. 执行方式说明

| 方式 | 用例 | 说明 |
|---|---|---|
| mano-cua | L1.1, L1.2, L1.3, L1.4 | VLA 模型驱动浏览器自动化,真实 GUI 交互 |
| AppleScript+JS | L1.4b, L1.5, L1.6, L1.7 | JS DOM 操作 + 文件下载验证 |
| 源码审计 | L1.8, L2.1–L2.8, L3.1–L3.7 | 直接审查部署代码逻辑,辅以 DOM 结构验证 |

**降级原因：** mano-cua 后端 (mano.mininglamp.com) 自 19:10 GMT+8 起持续故障（session 创建后 step API 返回 404 或无响应），L1.4b 尝试 4 次后降级。

---

## 5. 截图证据

所有截图保存于 `reports/singer-ntp/screenshots/`：

| 文件 | 说明 |
|---|---|
| L1.4b-save.png | 保存后页面状态 |
| L1.4b-restore.png | 刷新后恢复状态（title/content 互换） |
| L1.4b-reload-final.png | 最终恢复验证 |
| L1.5-dialog.png | 导出对话框（3 格式，TXT 默认） |
| L1.5-after-export.png | TXT 导出后 |
| L1.6-export-md.png | MD 导出 |
| L1.7-export-html.png | HTML 导出 |
| L2.1-after-dark-toggle.png | 暗色模式切换后（无变化） |
| L2.2-dark-persist.png | 暗色持久化验证 |

下载文件验证于 `~/Downloads/`：
- `ExportTitle.txt` (26 bytes) — 纯文本格式正确
- `ExportTitle.md` (28 bytes) — Markdown `# Title` 格式正确
- `ExportTitle.html` (175 bytes) — 完整 HTML 结构正确

---

## 6. 风险评估

| 风险 | 等级 | 说明 |
|---|---|---|
| 数据完整性 | **高** | BUG-001 导致保存后 title/content 互换，用户数据被破坏 |
| 可用性 | **高** | BUG-002 标题文字不可见，核心输入功能受损 |
| 主题切换 | **中** | BUG-003 暗色模式切换无效，但刷新后可恢复 |
| 字体功能 | **中** | BUG-004 字体映射全错，功能完全失效 |
| 国际化 | **低** | BUG-005 中文文件名被过滤，影响非 ASCII 用户 |

---

## 7. 建议

1. **BUG-001 [Critical]** 必须立即修复：交换 `localStorage.setItem` 的第二个参数
2. **BUG-002 [Major]** 修复 `.title-input` CSS `color` 属性
3. **BUG-003 [Major]** 取消注释 `setAttribute('data-theme', ...)` 行
4. **BUG-004 [Major]** 修正 `setFont()` 中的 CSS 变量映射
5. **BUG-005 [Minor]** 扩展 `sanitizeFilename()` 正则以支持中文字符

---

_报告生成时间: 2026-04-22 21:40 GMT+8_
_测试执行者: MOSS (550W)_
