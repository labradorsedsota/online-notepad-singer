# CHANGELOG — VLA-NTP (App 01 在线记事本)

## v1.0 (2026-04-21) — 验收通过

### 功能
- F-01: 笔记标题输入（Input, placeholder, maxlength=100）
- F-02: 内容编辑区（Card + textarea 自动扩展，min-height 300px）
- F-03: 保存到 localStorage（vla-ntp-* 前缀，成功/空内容提示）
- F-04: 导出（TXT/Markdown/HTML 三格式，Dialog + Radio）
- F-05: 清空（confirm 确认，空白状态提示）
- F-06: 暗色模式 Switch（持久化）
- F-07: 字体选择 Select（默认/等宽/衬线/无衬线，持久化）
- F-08: 按钮 Tooltip（300ms hover 延迟）
- F-09: 底部链接（关于/帮助 Dialog）
- F-10: 未保存提醒 Alert（编辑后显示，保存后消失）

### 组件覆盖（11/11）
Input, Button, Radio, Switch, Select, Dialog, Message, Alert, Tooltip, Link, Card

### 测试
- 22 条用例全部 PASS（L1×8 + L2×8 + L3×5 + BVA×1）
- 测试工具：mano-cua
- 测试执行人：Moss

### 部署
- GitHub Pages: https://labradorsedsota.github.io/online-notepad/
- 单 HTML 文件，零外部依赖

### 团队
- PM: Pichai | 开发: Fabrice | 测试: Moss
- 品鉴者: 廖雨亭 | 需求方: 廖雨亭
