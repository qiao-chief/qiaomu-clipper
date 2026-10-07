# 在 macOS 上部署乔木剪藏：踩坑记录

本文件是这个 Fork 自己加的，不属于上游项目。记录 2026 年 10 月在一台 Apple 芯片 Mac 上，以「加载已解压」方式安装扩展 1.14.3、本地保存助手和本机字幕引擎时遇到的问题和解决办法。

## 1. 装完本地助手后要重载扩展

现象：三连 Q 剪藏完全静默，既不落盘也不写剪贴板，而阅读模式和编辑器正常。助手端没有收到任何请求。

解决：在 `chrome://extensions` 重载扩展。`native/README.md` 写了这一步，容易跳过。

排查时可以先跑 `python3 native/install.py --check`，返回 `ok: true` 说明问题不在助手。

## 2. 本机字幕引擎安装到 80% 失败

现象：安装 `mlx-qwen3` 时报「模型下载失败」，错误是 `Using SOCKS proxy, but the 'socksio' package is not installed`。

原因：系统设置了 SOCKS 代理，模型下载会走它，但引擎的虚拟环境里没有 socks 支持。

解决：给引擎的虚拟环境补上依赖，再把模型补下来，不需要改系统代理。

```sh
~/.local/share/qiaomu-clipper/tools/mlx-qwen3/bin/pip install "httpx[socks]"
~/.local/share/qiaomu-clipper/tools/mlx-qwen3/bin/python -c \
  "from huggingface_hub import snapshot_download; snapshot_download('Qwen/Qwen3-ASR-0.6B')"
```

模型实际约 1.8 GB。补完后 `install.py --check` 的 `modelDownloadNeeded` 变为 `false`，设置页显示「已安装，随时可用」。

## 3. 界面出现奇怪的中文，是 Chrome 在自动翻译

现象：设置页出现「即将违约」「一般的」「读者」「特性」这类词；学习页的英文文字稿显示成中文，但「中文翻译」开关其实是关着的。

原因：扩展界面是英文时，Chrome 的网页翻译会把整个扩展页面翻掉，包括文字稿。看起来像扩展自己翻的，实际不是，译文也不会随剪藏保存。

解决：对扩展页面关闭 Chrome 的自动翻译（地址栏翻译图标 → 显示原文 → 一律不翻译此网站），并在扩展的常规设置里把语言改成简体中文。

## 4. Kimi Coding 计划的 key 要建自定义服务商

Coding 计划的 key 只认 `https://api.kimi.com/coding/v1`。预置的 Moonshot 选项指向开放平台，用这类 key 会返回 401。添加服务商时选「自定义」，填上面的地址即可。

该端点只接受 `temperature=1`。扩展的请求体不带 temperature，所以不受影响。

## 5. 日记配了模板时，学习笔记不可用

库的 `.obsidian/daily-notes.json` 配了日记模板时，助手按设计拒绝追加到当天日记。Obsidian 插件 Qiaomu AI RSS 的「边读边记」走的是 Obsidian 自己的日记接口，不受这个限制。

## 6. 用脚本触发三连键做测试时要小心

在自动化标签页里派发三次 `keydown` 可以无头触发剪藏，方便验证。但有一次落盘的是另一个正开着的页面，推测扩展剪的是当前激活的标签页（未验证）。有人在用浏览器时做这种测试，可能会剪到不相关的页面。

## 实测耗时

Apple 芯片，本机 Qwen3-ASR，直连本地助手：

| 来源 | 时长 | 总耗时 | 结果 |
|---|---|---|---|
| B 站中文视频 | 60 秒 | 39 秒 | 19 句 |
| YouTube 英文视频 | 19 秒 | 28 秒 | 7 句 |
