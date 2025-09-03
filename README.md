# 绿盟报告提取工具

本工具可自动解压绿盟扫描报告 ZIP 包，并递归解析其中的 HTML 报告文件，最终统一输出为 Excel 文件。


功能
1. 自动解压当前目录下的 zip（如：绿盟.zip）
2. 在当前目录与解压目录内递归查找 HTML 报告并解析
3. 统一提取并输出两个 Excel 文件：
   - 整理结果/漏洞报告.xlsx
     表头：序号, IP, 端口, 漏洞名称, 风险等级, 漏洞说明, 加固建议, CVE
   - 整理结果/开放端口.xlsx
     表头：序号, IP, 端口

环境要求
- Python >= 3.8
- 支持 Windows / Linux / macOS

依赖安装
1. 确认已经安装好 Python 与 pip
2. 在命令行运行以下命令安装依赖:
   pip install -r requirements.txt

说明:
- shutil、os、zipfile 等是 Python 内置库，不需要额外安装
- pandas, openpyxl, beautifulsoup4, lxml, tqdm 需要安装

使用方法
1. 克隆项目
   git clone https://github.com/zhkali127/nsfocus-report-extractor.git
   cd nsfocus-report-extractor

2. 安装依赖
   pip install -r requirements.txt

3. 将绿盟扫描报告压缩包 (如: 绿盟.zip) 放到当前目录

4. 运行脚本
   python main.py

5. 转换完成后，可以在 整理结果/ 目录下找到生成的 Excel 文件

作者信息
- 作者: zhkali127
- GitHub: https://github.com/zhkali127
