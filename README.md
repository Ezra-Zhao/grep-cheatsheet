# grep 命令速查表 · grep Cheatsheet

[![Stars](https://img.shields.io/github/stars/Ezra-Zhao/grep-cheatsheet?style=social)](https://github.com/Ezra-Zhao/grep-cheatsheet/stargazers) [![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE) ![Languages](https://img.shields.io/badge/languages-3-blue)

Support & more: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — templates & workbooks I use](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)


> 🌍 [Español](README.es.md) · [Português](README.pt.md)

一份**一页纸**的 grep 速查表：30 条真实可用的用法——日常搜索开关、递归与文件过滤、三种正则风味与"一行要匹配到什么程度"、上下文与输出整形，以及脚本真正依赖的管道行为（退出码、二进制文件、NUL 记录、实时流）。每个条目只讲三件事——**它是干什么的、一条能直接复制粘贴的命令、适用场景标签**。不编造语义：全部 30 条都在 GNU grep 3.11 上对着自造夹具目录树（双日志、笔记、C/Python 源码、二进制文件、NUL 分隔文件、符号链接、带空格文件名）**一条一条真实执行、输出与退出码核对过**才入表，语义逐项对照 GNU Grep 手册；GNU 与 BSD 的差异在表末尾专列。

## 面向谁

运维工程师、后端开发者、SRE，以及任何"日志有几百万行、只想知道哪几行出事了"、"脚本里 `grep` 返回值到底该怎么判"的人。如果你的搜索总是慢半拍——递归扫进不该扫的目录、二进制文件只回一句 matches、管道里输出迟迟不出来——这张表就是为你准备的。

## 怎么用

1. 直接看 **[CHEATSHEET.md](CHEATSHEET.md)** ——速查表正文，英文命令全球开发者都无压力。
2. 先分清你手里的 grep 是谁：`grep --version` 第一行会写 GNU grep 及版本；macOS 自带的是 BSD grep，没有 `-P`。表内 30 条在 GNU grep 3.11 上验证过，表末的实现差异注记请务必看一眼。
3. 按"基础 → 模式 → 上下文与输出 → 管道与脚本"的顺序，在自己的练习文件里每个条目亲手敲一遍，一小时就顺手了。

## 先跑起来（30 秒验证环境）

```bash
grep --version | head -1               # 先确认实现：GNU grep 及版本，BSD 差异表末尾专列
mkdir -p /tmp/proj && printf 'INFO ok\nERROR disk full\n' > /tmp/proj/app.log
grep -c 'ERROR' /tmp/proj/app.log      # 1——匹配行计数，这就是 grep 最常见的招牌动作
```

## 一条能直接用的黄金组合

```bash
grep -rn --include='*.log' 'ERROR' proj
```

递归＋行号＋只搜日志文件：整棵目录树里所有日志的 ERROR 行一次性带文件名和行号列出来，源码和二进制不碰。事故复盘的第一条命令，通常就是它。

## 收录了什么

| 分类 | 内容 |
|---|---|
| 🧱 基础 | 纯搜索 / -i 忽略大小写 / -v 反选 / -c 计数 / -n 行号 / -l 与 -L 文件清单 / -r 递归 / -R 跟随符号链接 / --include、--exclude、--exclude-dir 过滤（9 条） |
| 🎯 模式 | -E 扩展正则与 BRE 的反斜杠陷阱 / -F 定串字面匹配 / -w 整词 / -x 整行 / -e 多模式 / -f 模式文件 / ^ 与 $ 锚点（7 条） |
| 🖍️ 上下文与输出 | -A 后文 / -B 前文 / -C 双向与组块合并 / --color 高亮 / -o 只输出匹配部分 / -H 与 -h 文件名前缀控制（6 条） |
| 🔧 管道与脚本 | -q 与 0/1/2 退出码契约 / -s 静默报错 / -m 提前停 / 二进制三件套（提示、-a、-I） / -z NUL 记录与跨行匹配 / --line-buffered 实时管道 / -P PCRE 前后顾 / 以 - 开头的模式（8 条） |

官方文档只链 https://www.gnu.org/software/grep/manual/，所有命令均对照 GNU Grep 手册核实过语义，并在本机 GNU grep 3.11 上全部实跑通过；示例只在自造的夹具文件里操作，不涉及任何真实数据。

## 为什么是这 30 条

- **退出码契约必须上榜**：0 有匹配、1 没匹配、2 出错——脚本里一半的 grep 事故来自把"没找到"（1）当成"出错"，或者没料到 `-c` 打印 0 时退出码仍然是 1。表里逐项写死、逐项实测。
- **-r 与 -R 只差一笔，行为差一截**：`-r` 只跟随命令行上点名的符号链接，树里遇到的链接一律跳过；`-R` 全部跟随。不知道这条，递归搜索会静默漏掉链接背后的文件。
- **二进制文件的"假阴性"要标出来**：匹配行被压制、只在 stderr 留一句 `binary file matches`——只看 stdout 的人会以为没搜到。`-a` 强行打印、`-I` 干脆跳过，两种正解都给了。
- **-z 的跨行匹配有个隐形前提**：模式里的换行必须是真换行（bash 的 `$'…'`），普通引号里的 `\n` 不起作用——实测踩过才写进表里。

## 欢迎 PR

发现命令写错了、参数过时了、翻译不到位——欢迎提 Pull Request。新增条目请保持统一格式（一句话说明 / bash 代码块 / 标签），**只收真实可用的命令，不确定的宁可不收**。

## 📚 更多速查表

同一作者的其他速查表，同样有中/西/葡三语版本：

- [SSH 命令速查表](https://github.com/Ezra-Zhao/ssh-cheatsheet) — 28 条连接、密钥、隧道与加固命令
- [FFmpeg 命令速查表](https://github.com/Ezra-Zhao/ffmpeg-cheatsheet) — 28 条音视频处理命令
- [Git 实战速查表](https://github.com/Ezra-Zhao/git-cheatsheet) — 30 条日常+救命命令
- [Docker 命令速查表](https://github.com/Ezra-Zhao/docker-cheatsheet) — 28 条容器命令
- [curl 命令速查表](https://github.com/Ezra-Zhao/curl-cheatsheet) — 29 条 API 调试命令
- [正则表达式速查表](https://github.com/Ezra-Zhao/regex-cheatsheet) — 30 条实战正则模式
- [jq 命令速查表](https://github.com/Ezra-Zhao/jq-cheatsheet) — 30 条 JSON 处理 filter
- [终端神器速查表](https://github.com/Ezra-Zhao/terminal-superpowers-cheatsheet) — 14 个终端效率工具
- [Adobe 免费替代速查表](https://github.com/Ezra-Zhao/adobe-free-alternatives-cheatsheet) — 16 个 Adobe 免费替代品
- [退订省钱速查表](https://github.com/Ezra-Zhao/desubscribe-cheatsheet) — 10 组退订替代，年省约 $1,260
- [kubectl 命令速查表](https://github.com/Ezra-Zhao/kubectl-cheatsheet) — Kubernetes 常用命令速查
- [SQL 查询速查表](https://github.com/Ezra-Zhao/sql-cheatsheet) — 常用 SQL 查询与函数速查
- [Vim 命令速查表](https://github.com/Ezra-Zhao/vim-cheatsheet) — 30 条编辑、搜索替换与宏命令
- [tmux 命令速查表](https://github.com/Ezra-Zhao/tmux-cheatsheet) — 30 条会话、窗口与窗格命令
- [systemd 命令速查表](https://github.com/Ezra-Zhao/systemd-cheatsheet) — 30 条服务管理与日志命令
- [find + xargs 命令速查表](https://github.com/Ezra-Zhao/find-xargs-cheatsheet) — 30 条找文件与批量执行命令
- [awk 命令速查表](https://github.com/Ezra-Zhao/awk-cheatsheet) — 30 条字段处理、统计与双文件 join 命令

## License

MIT © 2026 Guangyi Zhao，详见 [LICENSE](LICENSE)。

📣 Telegram 频道：https://t.me/EzraZhao777（各平台的更新都同步到这里）
