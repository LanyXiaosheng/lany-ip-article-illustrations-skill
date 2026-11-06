# Lany IP Article Illustrations

完整阅读 Markdown 或长文，为关键判断、机制和工作流配上同一角色的正文插图。默认使用**像素小生、干净白底、16:9、1600×900**；图数按内容决定，通常为 1—8 张。

适合给长文解释机制、展示关键动作或拆解交付过程。角色沿用用户确认头像或仓库内置参考；这套技能不提供通用角色创建流程。

## 安装与更新

需要能读取本地 Skill 的 Agent；以下为 Codex 本地安装示例。仓库名带 `-skill`，实际调用名为 `$lany-ip-article-illustrations`。

```sh
mkdir -p ~/.agents/skills
git clone https://github.com/LanyXiaosheng/lany-ip-article-illustrations-skill.git ~/.agents/skills/lany-ip-article-illustrations
```

按照 [Codex 官方技能文档](https://learn.chatgpt.com/docs/build-skills)，用户级技能目录为 `~/.agents/skills/`，项目级可放在 `.agents/skills/`。安装后应自动发现；未显示时重启 Codex。若已有 `~/.codex/skills/lany-ip-article-illustrations` 且当前客户端能够加载，请沿用该副本，避免重复安装同名技能。其他 Agent 按其技能目录规则放入完整仓库，并让它读取 `SKILL.md`。

更新前先检查本地修改；有修改时先提交或备份，再拉取。以下路径要与实际安装位置一致：

```sh
git -C ~/.agents/skills/lany-ip-article-illustrations status --short
git -C ~/.agents/skills/lany-ip-article-illustrations pull --ff-only
```

仓库包含工作流和资源，不自带生图服务或模型凭据。只读方案/提示词可以直接使用；实际生图取决于宿主已提供的图片生成能力。

## 第一次使用

将文章附到对话中，或把下面的 `article.md` 换成实际路径：

```text
使用 $lany-ip-article-illustrations，完整读取 article.md，为正文配图。
角色使用我本次确认的头像；没有新头像时使用仓库内置像素小生。
请按文章内容决定图数，每张只解释一个观点。
默认白色背景、1600×900，先保存配图计划，再逐张生成并校验。
这次只生成图片，报告建议插入的段落，不修改原文。
```

### 配图并写回 Markdown

```text
使用 $lany-ip-article-illustrations，读取 article.md，生成3张正文插图并写入文章。
采用白色背景和同一像素小生，每张对应不同正文锚点。
所有正式图片落盘并通过检查后，先生成时间戳备份，再用相对路径插入。
保留已有图片、frontmatter、代码块和表格，不重复插入已有引用。
交付时列出正式图片、备份和修改后的文章路径。
```

### 指定背景

```text
使用 $lany-ip-article-illustrations，为 article.md 的核心工作流生成一张透明背景插图。
保持像素小生、品牌色和16:9构图；输出带有效Alpha的PNG。
在深色底和棋盘格上检查边缘，不能用白底图假装透明。
这次不写回文章。
```

深色背景请明确写“黑色/深色背景”。未指定时采用白色背景，同一篇文章保持一致的人物媒介。

## 输入与前置条件

| 输入/能力 | 要求 |
| --- | --- |
| 文章 | 可读取的完整 Markdown、附件或正文；多份候选文件时明确指定目标 |
| 身份参考 | 本次确认头像优先，其次为内置头像、轻量角色参考 |
| 图数 | 可指定；未指定时按认知密度判断，合并重复观点 |
| 背景 | 白色为默认；透明或深色需要明确要求 |
| 写回权限 | 明确说“写入 Markdown”才修改文章 |
| 生图能力 | 宿主提供可用图片生成工具；仓库不包含独立生图服务 |
| 后处理 | Python 3.9+、Pillow；轻量参考图脚本还需要 macOS 的 `sips` |

## 工作流程与交付

1. 读取全文，挑选不同的认知锚点，保存配图计划和提示词。
2. 每张选择“流程拆解”（3—5 个节点）或“核心动作”（一个动作和可见结果）。
3. 逐张生图，立即保存工具实际返回的源图。
4. 校验角色、观点、中文、背景和比例，生成正式 PNG。
5. 需要写回时，校验备份与图片后用相对路径插入对应段落。

对于 `article.md`，典型输出为：

```text
文章所在目录/
├── article.md
├── article.backup-YYYYMMDD-HHMMSS.md   # 要求写回时
└── .lany-ip-assets/illustrations/article/
    ├── illustration-plan.md
    ├── prompts/
    ├── 01-topic-source.png
    └── 01-topic.png
```

`*-source.png` 是生成源图，正式 `01-topic.png` 才用于文章。同名产物增加版本后缀。交付按文章顺序列出模式、观点、图片路径和插入位置。

## 可选：手动运行后处理脚本

以下命令在已安装技能的机器上运行。把路径改为实际源图和输出位置；输出使用新文件名，脚本本身不会替你防止覆盖。

### 准备 Python 环境

```sh
python3 -m venv .venv-lany-images
. .venv-lany-images/bin/activate
python -m pip install Pillow
```

### 裁切为正式尺寸并校验

```sh
python ~/.agents/skills/lany-ip-article-illustrations/scripts/finalize_ratio.py   ./01-topic-source.png ./01-topic.png --size 1600x900
python ~/.agents/skills/lany-ip-article-illustrations/scripts/validate_output.py   ./01-topic.png --size 1600x900 --background white
```

`finalize_ratio.py` 按目标比例裁切并缩放，默认中心裁切。可用 `--focal-x` 和 `--focal-y`（0—1）调整焦点；先查看构图，避免裁掉人物或标签。

`validate_output.py` 检查尺寸；透明模式另检查 Alpha 通道，深色模式检查近白像素占比。白底均匀、人物身份、中文准确性和真正透明的边缘仍须目视检查；Alpha 通道存在不等于背景已透明。

### 生成轻量身份参考（仅 macOS）

```sh
python ~/.agents/skills/lany-ip-article-illustrations/scripts/prepare_reference.py   ./avatar.png ./avatar-lite.png --max-long-side 384
```

此脚本依赖系统 `sips`；其他系统可直接使用仓库内的轻量参考，或由宿主图像工具生成。

### 每批生图前的代理预检

```sh
python ~/.agents/skills/lany-ip-article-illustrations/scripts/imagegen_preflight.py   --host 127.0.0.1 --port 7897 --timeout 8
```

当前 Skill 要求每批生图前执行预检，默认检查本地代理 127.0.0.1:7897；端口按实际配置修改。若你的环境不走本地代理，需要先向 Agent 说明实际网络链路并调整预检流程，不能把失败结果视为已通过。该脚本检查代理端口与固定图片端点的握手，不提供认证，也不能证明图片上传或生成成功；401/403等可表示端点可达。宿主的实际工具返回才是生图成功依据。

## 常见问题

- **图片数量太多：** 在请求中给出数量或上限，并要求每张承担不同的理解任务；不要按章节机械配图。
- **角色变了：** 附上已确认的头像并明确它是身份真源；同一文章复用同一参考。
- **只有计划没有图片：** 检查宿主生图工具是否可用。失败时保留已成功图片和计划，恢复后继续剩余部分。
- **图片有白底却要求透明：** 使用透明背景模式，检查有效透明像素和实际边缘，不能只看文件扩展名。
- **图片生成了但文章没变：** 确认明确要求写回；任一正式图片未落盘或未通过验收时，工作流不修改 Markdown。
- **Python 报 `No module named PIL`：** 在运行脚本的同一虚拟环境安装 Pillow。
- **一张图塞太多数字：** 数值比较交给 [lany-data-charts-skill](https://github.com/LanyXiaosheng/lany-data-charts-skill)，精确节点关系可交给 [lany-infographics-skill](https://github.com/LanyXiaosheng/lany-infographics-skill)。

## 仓库导航

- [SKILL.md](SKILL.md)：完整执行规则。
- [agents/openai.yaml](agents/openai.yaml)：技能展示信息。
- [文章工作流](references/article-workflow.md)：数量、锚点与提示词。
- [品牌体系](references/brand-system.md)与[角色规范](references/character-spec.md)。
- [生图可靠性](references/imagegen-reliability.md)：预检与失败处理。
- [DESIGN.md](DESIGN.md)：拆分设计与边界。

来源：[lany-skills/lany-ip-article-illustrations](https://github.com/LanyXiaosheng/lany-skills/tree/master/lany-ip-article-illustrations)。本仓库独立安装，调用名保持 `lany-ip-article-illustrations`。
