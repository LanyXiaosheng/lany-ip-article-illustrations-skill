# lany-ip-article-illustrations-skill

完整读取 Lany 小生的 Markdown 或长文，自动判断正文配图数量，以固定像素小生角色和咨询报告视觉逐张生成默认白色背景的 16:9 正文插图；只有用户明确要求透明或深色背景时才切换背景模式，并在用户要求时备份原文后写入相对路径。用于 Lany 小生的文章配图，不用于通用角色或自由画风任务。

## 独立仓库安装

本仓库包含 `lany-ip-article-illustrations` 技能及其参考资料与资源，支持独立安装。依赖可读取 SKILL.md 的 Agent；实际生图需要宿主提供图片生成工具。

```sh
git clone https://github.com/LanyXiaosheng/lany-ip-article-illustrations-skill.git ~/.codex/skills/lany-ip-article-illustrations
```

若目标目录已存在，请先检查其中的自定义内容。安装后重新加载技能，调用 `$lany-ip-article-illustrations`。完整工作流见 [SKILL.md](SKILL.md)。

来源：[lany-skills/lany-ip-article-illustrations](https://github.com/LanyXiaosheng/lany-skills/tree/master/lany-ip-article-illustrations)。技能名称保持不变。

## 脚本依赖

图像处理与校验脚本需要 Python 3.9+ 和 Pillow（`python3 -m pip install Pillow`）；`prepare_reference.py` 使用 macOS 自带的 `sips`。`imagegen_preflight.py` 可选检查本地代理，默认端口 7897，可通过参数调整。脚本输出路径由调用者指定，使用新文件名以保留原图。
