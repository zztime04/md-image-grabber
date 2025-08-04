# 结构设计

用于指导后续代码实现。包括目录布局、模块划分、主要类与函数职责，以及数据流概览。

---

## 一、目录布局

```text
md-image-grabber/
├── README.md
├── setup.py                # （可选）打包安装脚本
├── requirements.txt        # 依赖列表
├── source/
│   ├── __init__.py
│   ├── cli.py              # 命令行接口
│   ├── scanner.py          # 文件/目录扫描
│   ├── parser.py           # Markdown 解析与 AST 操作
│   ├── downloader.py       # 资源下载
│   ├── writer.py           # AST 渲染与输出
│   └── utils.py            # 通用工具函数（路径、日志、URL 处理等）
└── tests/
    ├── test_scanner.py
    ├── test_parser.py
    ├── test_downloader.py
    └── test_writer.py
```

---

## 二、模块职责

### 1. `cli.py`

* 定义 `--input/-i`、`--output-dir/-o`、`--verbose` 等参数。
* 解析命令行选项并初始化全局配置对象。
* 调用核心流程：扫描 → 解析 → 下载 → 重写 → 写出。

### 2. `scanner.py`

* 函数 `scan_files(input_path: Path) → List[Path]`

  * 单文件或目录模式：识别所有 `.md` 文件（递归）。
  * 返回源文件的绝对路径列表。

* 函数 `compute_output_path(src: Path, input_root: Path, output_root: Path) → Path`

  * 基于源文件相对路径，计算对应的输出文件路径。

### 3. `parser.py`

* 类 `MarkdownImageParser`

  * 初始化：接收文件内容。
  * 方法 `extract_image_nodes() → List[ImageNode]`：遍历 AST，收集原始 URL、alt 文本及在 AST 中的引用位置。
  * 方法 `update_image_sources(mapping: Dict[orig_url, local_path]) → AST`：在原 AST 上替换节点。

* 数据结构 `ImageNode`（例如 `namedtuple`）

  * `url: str`, `alt: str`, `position: SourcePosition`

### 4. `downloader.py`

* 函数 `download_image(url: str, dest: Path, timeout: int, retries: int) → bool`

  * 流式下载、保存到 `dest`，返回成功标志。

* 可选并发封装，如 `download_images_concurrent(tasks: List[DownloadTask])`.

* 数据结构 `DownloadTask`

  * `url: str`, `dest: Path`

### 5. `writer.py`

* 函数 `render_ast_to_markdown(ast: AST) → str`

  * 利用解析库将修改后的 AST 渲染回 Markdown 文本。
* 函数 `write_file(path: Path, content: str)`

  * 创建父目录、写入文本文件。

### 6. `utils.py`

* 路径与文件名处理：

  * `sanitize_filename(name: str) → str`
  * `ensure_unique(path: Path) → Path`（重复时加序号或哈希）
* URL 解析：

  * `get_basename_from_url(url: str) → str`
* 日志：简单的 `log_info()`, `log_warn()`, 根据 `--verbose` 决定输出。

---

## 三、核心数据流

1. **CLI** 读取参数，构造 `Config(input_root, output_root, verbose)`。
2. **Scanner** 根据 `input_root` 列出所有源 `.md` 文件。
3. 对每个 **源文件**：

   1. **Parser** 读入文件内容 → 构建 AST → `extract_image_nodes()` 得到所有外部图片 URL 列表。
   2. 根据文件在 `output_root` 下创建对应子目录，以及该目录内的 `images/` 文件夹。
   3. **Downloader** 遍历 URL 列表，将图片下载到 `images/`，返回 `url→local_path` 映射。
   4. **Parser** 调用 `update_image_sources(mapping)` 替换 AST 中的 URL。
   5. **Writer** 将修改后的 AST 渲染为 Markdown 文本，写入 `output_root` 对应的 `.md` 文件。

---

## 四、异常与扩展点

* **异常捕获**：各模块应捕获并记录自身异常，不影响整体流程。
* **并发下载**：在 `downloader.py` 中添加可选线程池实现。
* **增量更新**：`scanner.py` 可比较源文件与目标文件时间戳，跳过未修改文件。
* **配置文件支持**：可在 `cli.py` 增加读取 YAML/JSON 配置的逻辑，传递至各模块。

---

此结构清晰地将各项功能职责解耦，便于测试和扩展。下一步可以根据此架构开始编写对应模块与测试用例。
