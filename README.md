# dsh-plugin-index

> DSH 插件市场索引 —— 给 [DeepSeek Harness](https://github.com/) 的插件市场（Discover）提供一个可安装的插件清单。

## 这个仓库是什么

DSH 的插件市场**不是官方收稿平台**，而是**可配置的索引源**：用户在设置里填任意 http(s) 索引 URL，市场读它来展示和安装插件。

这个仓库就是这样一个索引。里面有两个 macOS 优先的插件。

## 怎么用

在 DSH 里：**设置 → 插件 → 插件市场（Discover）**，索引地址填：

```
https://raw.githubusercontent.com/huu8/dsh-plugin-index/main/index.json
```

然后就能在市场里看到并安装下面两个插件。

## 包含的插件

### [aoic-panel-mac](https://github.com/huu8/aoic-panel-mac)

AOCI 仓库认知面板。把 AOCI 的仓库认知索引做成 DSH 标签页：

- 索引正文浏览与搜索（按文件名、标签、复杂度、S 字段）
- 治理体检卡——把良性发现（`code_skipped`）与真正的阻塞项分开，避免 443 条全标红看不出重点
- S 红线一键筛选
- 一键 verify / maintain
- 索引在线编辑 + 自动备份恢复

上游 `dsh-aoci-panel` 仅支持 Windows，在 macOS 上治理功能全部失效（二进制探测只认 `.exe`、home 只读 `USERPROFILE`、`install.ps1` 需要 pwsh）。这个版本修好平台适配并补齐能力。

### [codegraph-panel-mac](https://github.com/huu8/codegraph-panel-mac)

CodeGraph 认知面板。**CodeGraph 官方没有 UI**（`codegraph ui` 仍报 `coming in an upcoming release`），这个插件补上：

- 索引概览（文件/节点/边/体积/语言/待同步 + 类型分布）
- 符号搜索
- **cytoscape 交互式调用关系图**（以符号为中心 1–3 跳邻域，节点按类型着色、边按关系类型着色，可缩放拖拽）
- 影响面分析（改一处会波及什么，按跳数分组）
- 符号详情（元数据 + 源码与调用链）
- 一键 sync / 重建索引

图数据直读 `.codegraph` 的 `nodes` / `edges` 表（因为 CLI 的 `impact` 输出只有平铺节点、没有连线，画不出图），一律只读打开。

## 索引格式

遵循 DSH 官方 `dsh-plugin-market-index` schema（`format: 1`）。结构概要：

```jsonc
{
  "format": 1,
  "title": { "zh": "…", "en": "…" },
  "description": { "zh": "…", "en": "…" },
  "metadata": {
    "categories": { "<id>": { "title": { "zh": "…", "en": "…" } } },
    "labels":     { "<id>": { "title": { "zh": "…", "en": "…" } } },
    "sortKeys":   { "<id>": { "title": { "zh": "…", "en": "…" }, "order": "asc" | "desc" } }
  },
  "plugins": [
    {
      "name": "包名",
      "title": { "zh": "…", "en": "…" },
      "description": { "zh": "…", "en": "…" },
      "author": "…",
      "homepage": "https://…",
      "repository": { "url": "https://…" },
      "categories": ["<category id>"],
      "labels": ["<label id>"],
      "keywords": ["…"],
      "engines": { "dsh": ">=0.2.0-rc.1" },
      "versions": [
        {
          "version": "1.0.0",
          "publishTimestamp": 1794290898,
          "source": {
            "type": "git",
            "url": "https://github.com/<owner>/<repo>.git",
            "commit": "<40 位 commit sha>"
          }
        }
      ]
    }
  ]
}
```

注意事项：

- 顶层必须有 `format: 1`（不是 `version`）
- `categories` / `labels` 里用到的 id 必须在 `metadata` 里声明过，否则校验报 `references undeclared id`
- `source.commit` 必须是完整的 40 位 sha，市场按它锁定可复现版本
- `publishTimestamp` 是 Unix 秒（整数）

## 校验

改动后可以用 DSH 自带的校验器验证（路径按你的安装调整）：

```bash
node -e "import('/Applications/DeepSeek Harness.app/Contents/Resources/app.asar.unpacked/dsh/node_modules/@deepseek-ai/dsh-plugin-market-index/lib/index.js').then(m => { const idx = JSON.parse(require('fs').readFileSync('index.json','utf8')); m.parseMarketIndex(idx, 'https://raw.githubusercontent.com/huu8/dsh-plugin-index/main/index.json'); console.log('✓ 通过'); })"
```

校验失败会给出精确路径，例如 `$.plugins[0].versions[0].source.commit must be a string with a non-whitespace character`。

## 许可

MIT
