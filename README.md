grab-cli 需求文档

1. 项目概述

grab-cli 是一个面向 Windows 环境的本地文本与结构化数据抓取、过滤工具。它以命令行方式运行，不依赖 GUI，不依赖网络服务，主要面向本地文件、标准输入和结构化文本数据。

核心定位：

- 支持类似 grep 的关键词行过滤；
- 支持基于字段/列的条件过滤；
- 支持正则、文本、数值、相等比较等多种过滤方式；
- 支持 CSV/TSV 等分隔符数据的结构化过滤；
- 支持标准输入管道；
- 支持结果统计、导出和格式化输出；
- 面向 Windows 10/11 环境，使用 C++17、CMake 和 vcpkg 构建。

2. 目标用户

- 需要在 Windows 本地快速筛选文本、日志、CSV 数据的用户；
- 希望比传统关键词搜索更结构化地按字段过滤数据的用户；
- 学习 C++、CMake、vcpkg、命令行工具开发的开发者。

3. 运行环境

- 操作系统：Windows 10 / 11
- 语言标准：C++17
- 构建系统：CMake
- 依赖管理：vcpkg
- 推荐编译器：MSVC（Visual Studio 2022）或 MinGW
- 运行方式：CLI，无 GUI
- 输入来源：文件、标准输入
- 输出目标：控制台、文件
- 路径处理：使用 std::filesystem，避免硬编码路径分隔符
- 文件读写：使用 std::fstream、std::cin、std::cout
- 编码建议：默认按 UTF-8 文本处理，避免中文内容乱码

4. 核心功能

4.1 行级关键词过滤

支持对整行文本进行关键词匹配，行为类似 grep。

示例：

grab "error" log.txt
grab -i "warning" log.txt

要求：

- 支持单关键词匹配；
- 支持忽略大小写；
- 支持显示行号；
- 支持统计匹配行数；
- 支持多文件输入。

4.2 字段级过滤

支持按字段名或列索引进行过滤，适用于 CSV、TSV 等结构化文本。

示例：

grab -H -d ',' -f name --contains "Alice" data.csv
grab -H -d ',' -f score --gt 80 students.csv
grab -H -d ',' -f status --eq active data.csv

要求：

- 支持按列名过滤；
- 支持按列索引过滤；
- 支持指定分隔符；
- 支持识别表头；
- 支持多字段组合过滤。

4.3 过滤操作类型
操作   说明   示例
--contains   字段包含某字符串   --contains "error"
--eq   字段等于某值   --eq active
--ne   字段不等于某值   --ne failed
--gt   字段大于某值   --gt 80
--lt   字段小于某值   --lt 60
--gte   字段大于等于某值   --gte 60
--lte   字段小于等于某值   --lte 100
--regex   字段匹配正则表达式   --regex "err.*timeout"

4.4 正则过滤

支持使用正则表达式对字段或整行进行匹配。

示例：

grab --regex "err(or|ors)" log.txt
grab -f email --regex "@example.com" users.csv

要求：

- 支持基础正则匹配；
- 支持忽略大小写；
- 支持字段正则匹配；
- 支持整行正则匹配。

4.5 多条件组合

支持同时应用多个过滤条件，所有条件默认以“且”关系生效。

示例：

grab -H -d ',' -f status --eq active -f score --gte 60 data.csv

含义：

- status 字段等于 active；
- score 字段大于等于 60；
- 两个条件同时满足才保留该行。

4.6 标准输入支持

支持通过管道读取数据。

示例：

type data.csv | grab -H -d ',' -f age --lt 25
type log.txt | grab "error"

要求：

- 未指定文件时从标准输入读取；
- 支持与其他过滤选项组合使用；
- Windows 环境下可使用 type 或 PowerShell 管道输入。

4.7 输出控制

支持控制输出内容、格式和目标。

示例：

grab -n "TODO" src/main.cpp
grab -c "error" log.txt
grab -H -f score --gt 80 data.csv -o result.csv

要求：

- 支持显示行号；
- 支持只输出匹配行数；
- 支持导出到文件；
- 支持保留表头输出；
- 支持高亮匹配内容。

5. 命令行接口设计

5.1 基本格式

grab [OPTIONS] [PATTERN] [FILE...]

5.2 位置参数
参数   说明
PATTERN   关键词或正则表达式，用于行级过滤
FILE   输入文件，可指定多个文件

5.3 常用选项
选项   说明
-f, --field   指定字段名或列索引
-d, --delimiter   指定分隔符，默认逗号
-H, --header   第一行作为表头
--contains   字段包含某字符串
--eq   字段等于某值
--ne   字段不等于某值
--gt   字段大于某值
--lt   字段小于某值
--gte   字段大于等于某值
--lte   字段小于等于某值
--regex   使用正则匹配
-i, --ignore-case   忽略大小写
-n, --line-number   显示行号
-c, --count   只输出匹配行数
-o, --output   将结果导出到文件
--highlight   高亮匹配内容
-h, --help   显示帮助信息
-v, --version   显示版本信息

6. 数据模型

6.1 行数据

每一行数据表示为一个字符串或字段列表：

struct Row {
    std::vector fields;
    std::string raw_line;
    int line_number;
};

6.2 过滤条件

struct Filter {
    std::string field;
    std::string op;
    std::string value;
    bool ignore_case = false;
};

6.3 匹配结果

struct MatchResult {
    int line_number;
    std::string raw_line;
    std::vector fields;
    bool matched;
};

7. 处理流程

1. 解析命令行参数；
2. 确定输入来源：文件或标准输入；
3. 按行读取数据；
4. 根据分隔符拆分字段；
5. 识别表头；
6. 对每一行应用过滤条件；
7. 保留满足所有条件的行；
8. 按指定格式输出结果；
9. 可选导出到文件。

8. 项目结构

grab-cli/
├── CMakeLists.txt
├── vcpkg.json
├── README.md
├── include/
│   ├── grabber.hpp
│   ├── filter.hpp
│   ├── parser.hpp
│   └── printer.hpp
├── src/
│   ├── main.cpp
│   ├── grabber.cpp
│   ├── filter.cpp
│   ├── parser.cpp
│   └── printer.cpp
└── tests/
    └── grab_test.cpp

9. 依赖规划
依赖   用途
fmt   格式化输出
re2   正则表达式匹配
CLI11   命令行参数解析
Catch2   单元测试

vcpkg 配置示例：

{
  "name": "grab-cli",
  "version": "0.1.0",
  "dependencies": [
    "fmt",
    "re2",
    "cli11",
    "catch2"
  ]
}

Windows 环境下安装依赖示例：

vcpkg install fmt:x64-windows re2:x64-windows cli11:x64-windows catch2:x64-windows

CMake 构建示例：

cmake -B build -S . -DCMAKE_TOOLCHAIN_FILE=<vcpkg路径>/scripts/buildsystems/vcpkg.cmake
cmake --build build --config Release

10. 版本规划

v0.1.0

- 支持关键词行过滤；
- 支持文件输入；
- 支持忽略大小写；
- 支持显示行号；
- 支持基本帮助信息。

v0.2.0

- 支持标准输入；
- 支持多文件输入；
- 支持匹配行数统计；
- 支持简单输出格式化。

v0.3.0

- 支持分隔符解析；
- 支持表头识别；
- 支持字段级过滤；
- 支持 contains、eq、ne 操作。

v0.4.0

- 支持数值比较；
- 支持正则过滤；
- 支持多条件组合；
- 支持结果导出。

v0.5.0

- 支持匹配高亮；
- 支持单元测试；
- 完善错误提示；
- 完善帮助文档。

11. 非功能需求

- 不依赖网络；
- 不依赖 GUI；
- 不依赖复杂数据结构；
- 不依赖 Linux 专属 API；
- 路径处理使用 std::filesystem；
- 文件读写使用标准 C++ 库；
- 代码结构清晰，便于扩展；
- 错误信息明确；
- 支持常见文本文件和 CSV 文件；
- 支持 Windows 10/11 环境构建和运行。

12. 示例场景

日志筛选

grab "error" app.log
grab -i "warning" app.log
grab -n "timeout" app.log

Windows 管道示例：

type app.log | grab "error"

CSV 数据过滤

grab -H -d ',' -f score --gt 80 students.csv
grab -H -d ',' -f name --contains "Li" students.csv
grab -H -d ',' -f status --eq active -f score --gte 60 data.csv

管道处理

type data.csv | grab -H -d ',' -f age --lt 25
type log.txt | grab "error" | grab -c

结果导出

grab -H -d ',' -f score --gt 90 students.csv -o top_students.csv

13. 验收标准

- 能够在 Windows 10/11 上通过 CMake + vcpkg 成功构建；
- 能够正确读取文件并输出匹配行；
- 能够正确处理标准输入；
- 能够按字段进行过滤；
- 能够处理文本、数值、正则条件；
- 能够组合多个过滤条件；
- 能够输出匹配行数；
- 能够将结果导出到文件；
- 命令行参数解析稳定；
- 基本错误输入不会导致程序崩溃；
- 路径处理兼容 Windows 文件系统。
