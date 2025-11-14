## 说明

演示一个C++项目的基本组成，演示cmake以及gtest的基本使用方法。

## 注意事项

在编译后运行程序会报加载动态库失败的问题，这是因为可执行程序找不到我们编译的动态库导致的。

将所有依赖的 DLL（如 build/libhellolib.dll、build/libworldlib.dll）复制到 build/tests 或 build 目录下，确保 test\_hello.exe 能找到它们。

## build in Linux

```
mkdir build
cd build
```

执行cmake 生成 makefile

```
cmake .. -L -DCMAKE_SKIP_RPATH=FALSE
make
```

执行测试用例

执行 ctest 或者 make test或者进入tests目录 ./test_hello（可查看gtest详细日志）

## 在Windows下跨平台编译：

```
cmake -B build
cmake --build build
```

## 生成代码覆盖率报告

**步骤 1：准备依赖**
- 安装 `gcovr`。Windows 用户可以执行 `pip install gcovr`（推荐在本仓库的虚拟环境中）；MSYS2 用户也可以通过 `pacman -S gcovr` 安装。
- 确保使用 GCC/Clang 工具链；MSVC 暂不支持当前脚本中的 `--coverage` 选项。

**步骤 2：配置覆盖率构建目录**

Windows 下推荐使用 Ninja 生成器（需要将 GCC 工具链添加到 PATH）：
```powershell
# 将 w64devkit 或 MinGW 添加到 PATH 最前面
$env:Path = "C:\w64devkit\bin;" + $env:Path

# 清理旧的 build 目录（如果存在）
Remove-Item -Recurse -Force build

# 配置 CMake
cmake -S . -B build -G Ninja -DBUILD_TEST=ON -DENABLE_COVERAGE=ON -DCMAKE_BUILD_TYPE=Debug
```

Linux/macOS 示例：
```bash
cmake -S . -B build -G Ninja -DBUILD_TEST=ON -DENABLE_COVERAGE=ON -DCMAKE_BUILD_TYPE=Debug
```

**步骤 3：编译项目**
```powershell
cmake --build build
```

**步骤 4：运行测试并生成覆盖率报告**
```powershell
cmake --build build --target coverage
```

**步骤 5：查看报告**

该目标会自动执行 `ctest` 并在 `build/coverage` 目录下生成以下文件：
- `index.html` - HTML 主报告（在浏览器中打开查看完整报告）
- `index.<source>.html` - 各源文件的详细覆盖率页面
- `coverage.xml` - Cobertura XML 格式（供 CI 工具使用）
- `summary.txt` - 文本格式摘要
- `lcov.info` - LCOV 格式（可上传到 Coveralls、SonarQube 等分析平台）

打开 HTML 报告：
```powershell
# Windows
start build\coverage\index.html

# Linux/macOS
open build/coverage/index.html  # macOS
xdg-open build/coverage/index.html  # Linux
```

**CI 自动化**

GitHub Actions（`.github/workflows/coverage.yml`）会在 push/pull request 时自动运行上述流程，并将 `build/coverage` 作为构建产物上传，方便在 CI 中下载 HTML/LCOV 报告。
