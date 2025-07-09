

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

