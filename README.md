# nsfocus-report-extractor — HTML 报告解析与漏洞/端口导出工具

> 从各种扫描器（如国产/第三方扫描器导出的 HTML / ZIP 包）中提取漏洞信息与开放端口，生成统一的 Excel 报表（`漏洞报告.xlsx`、`开放端口.xlsx`）。

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE) [![Python](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org)

仓库：

* GitHub: `https://github.com/ouwenjin/nsfocus-report-extractor`
* Gitee: `https://gitee.com/zhkali/nsfocus-report-extractor`

---

## 简介

该脚本用于解析目录（或 ZIP 包）中的 HTML 报告（例如扫描器导出的报告页面或内嵌数据），通过多种启发式方法提取：

* 漏洞条目：IP、端口、漏洞名称、风险等级、漏洞说明、加固建议、CVE
* 开放端口清单：IP + 端口

支持对 HTML 中嵌入的 JSON/JS 对象优先解析（例如 `window.data`、内嵌 `vul` 列表），如果找不到则回退到表格和文本块的启发式解析，具有较强的容错性。

---

## 主要功能

* 自动解压目录下的 ZIP（可包含多个 HTML 报告），并解析其中的 HTML 文件
* 优先解析嵌入的 JSON/JS 数据（如 `window.data` / `vuls` 等结构）以获取结构化漏洞信息
* 回退解析表格（`<table>`）和文本块（`<div>`/`<p>`/`<li`>）以提取端口与漏洞
* 自动合并去重并按序号导出：`漏洞报告.xlsx`（含字段：序号、IP、端口、漏洞名称、风险等级、漏洞说明、加固建议、CVE）和 `开放端口.xlsx`
* 对常见字段进行多语言/多别名映射（支持中/英文表头变体）
* 支持 `tqdm` 进度显示（若未安装则降级为普通遍历）
* 输出路径与输入目录可通过 CLI 指定，支持备份已存在的输出文件

---

## 要求

* Python 3.8+
* 推荐在虚拟环境中运行（venv / conda）

依赖（示例）：

```
pandas>=1.3
beautifulsoup4>=4.9
lxml
```

> 脚本对 `pandas` 与 `beautifulsoup4` 有开箱依赖检查并在缺失时给出可读错误提示。若没有安装 `tqdm` 则不会影响功能，只是进度条降级。

---

## 安装

```bash
git clone https://github.com/ouwenjin/nsfocus-report-extractor.git
cd nsfocus-report-extractor
python -m venv .venv
source .venv/bin/activate    # Linux / macOS
.venv\Scripts\activate     # Windows
pip install -r requirements.txt
```

（`requirements.txt` 示例见仓库根或由作者提供）

---

## 使用说明

```
usage: rsas2_html_extractor.py [-h] [-i INPUT] [-o OUTPUT] [--force-regenerate | --no-force-regenerate]
                               [--no-unicode] [--margin MARGIN] [--pad PAD]

解析 HTML 报告并生成 漏洞报告.xlsx 和 开放端口.xlsx，并打印作者横幅
```

常用参数：

* `--input, -i`：输入目录，默认当前目录 `.`
* `--output, -o`：输出目录，默认 `./整理结果`
* `--force-regenerate` / `--no-force-regenerate`：强制重新解析（默认开启），关闭则若目录中已有输出文件会直接复用
* `--no-unicode`：在不支持 Unicode 的终端使用 ASCII 边框
* `--margin` / `--pad`：控制终端横幅显示的左侧外边距与内部填充

示例：

```bash
# 解析当前目录并输出到 ./整理结果
python rsas2_html_extractor.py

# 指定输入目录并将结果输出到 ./out
python rsas2_html_extractor.py -i ./reports -o ./out --force-regenerate
```

---

## 输出文件

脚本会在指定输出目录下生成：

* `漏洞报告.xlsx` — 列：`序号, IP, 端口, 漏洞名称, 风险等级, 漏洞说明, 加固建议, CVE`
* `开放端口.xlsx` — 列：`序号, IP, 端口`

如果输出文件已存在，脚本会先备份原始文件（在同目录生成带时间戳的 `_backup_` 文件），然后写入新文件。

---

## 解析策略（实现要点）

1. **优先 JSON/JS**：脚本会尝试从文件中提取以 `window.data` 或包含 `vuls`、`vul_items` 等标识的 JS 对象并 `json.loads` 解析（对常见尾随逗号做宽容处理）。
2. **结构化数组遍历**：若 JSON 解析成功，会遍历可能的 `vuls` / `vul_items` 列表，按常见字段抽取 IP/端口/CVE/描述/修复建议等。
3. **回退到表格**：若无法解析 JSON，则遍历 `<table>` 标签并基于表头映射（多种别名）提取字段；表格既可以是整齐的 `th`+`tr`，也能容忍一些不规则布局。
4. **文本启发式提取**：在 `<div>`/`<p>` 等文本块中查找 IP、端口、CVE、严重度关键词，作为补充数据来源。
5. **后处理归一化**：统一字段名称、风险等级映射、CVE 格式化，并去重后排序写入 Excel。

---

## 定制与扩展

* **字段别名**：可通过 `HEADER_MAP` 在脚本中增加更多表头别名以提高表格解析的识别率。
* **风险映射**：`normalize_risk()` 可按团队策略调整风险等级归一化逻辑（例如将数字评分映射为高/中/低）。
* **支持更多输入**：可扩展 `find_files()` 与 `unzip_all()`，加入对 `.nessus` 、`.xml` 等格式的支持。
* **导出格式**：目前导出为 Excel，可扩展为 CSV、JSON、或直接入库的功能。

---

## 常见问题与排错

* **未能解析任何漏洞**：请确认输入文件确实包含漏洞信息或内嵌数据。尝试打开其中一个 HTML 检查是否有 `vuls` / `window.data` 或清晰的表格。
* **编码问题**：脚本优先以 UTF-8 读取；若文件为其他编码请先转换或调整脚本的读取逻辑。
* **依赖缺失**：若提示 `pandas` 或 `beautifulsoup4` 未安装，请按 README 安装依赖。

---

## 贡献 & 许可证

欢迎提交 Issue / PR。 MIT 许可证。

---

## 作者

作者：`zhkali`
仓库：

* `https://github.com/ouwenjin/nsfocus-report-extractor`
* `https://gitee.com/zhkali/nsfocus-report-extractor`
