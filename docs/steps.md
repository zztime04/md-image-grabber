# 生成程序步骤

以下步骤指导如何利用 Codex 按照设计与结构实现 `md-image-grabber` 项目。

1. **初始化仓库结构**  
   - 根据 `docs/struct.md` 中的目录布局创建 `src/` 和 `tests/` 等目录，并添加空模块。  
   - 确保 `src` 包含 `cli.py`, `scanner.py`, `parser.py`, `downloader.py`, `writer.py`, `utils.py`。

2. **编写命令行接口**  
   - 在 `cli.py` 中使用 `argparse` 实现 `--input/-i`、`--output-dir/-o`、`--verbose` 参数。  
   - 定义 `Config` 数据类，串联后续流程。

3. **实现扫描模块**  
   - 在 `scanner.py` 编写 `scan_files` 和 `compute_output_path`，递归扫描 `.md` 文件并计算输出路径。

4. **实现解析模块**  
   - 在 `parser.py` 创建 `MarkdownImageParser` 类，使用 `markdown-it-py` 解析 Markdown 并提取图片节点。  
   - 提供 `extract_image_nodes` 和 `update_image_sources` 方法。

5. **实现下载模块**  
   - 在 `downloader.py` 定义 `DownloadTask` 和 `download_image`，使用 `requests` 下载图片至对应 `images/` 子目录。  
   - 可封装并发下载以提升效率。

6. **实现渲染与写入**  
   - 在 `writer.py` 使用解析库渲染修改后的 AST 为 Markdown，并通过 `write_file` 写入输出目录。

7. **编写通用工具**  
   - 在 `utils.py` 提供文件名清理、唯一性保证、URL 解析、简单日志等函数。

8. **串联流程**  
   - 在 `cli.py` 或单独的入口函数中按照“扫描 → 解析 → 下载 → 替换 → 写出”的顺序处理每个文件。

9. **编写并运行测试**  
   - 在 `tests/` 中为各模块编写单元测试，运行 `pytest` 确认功能正确。
