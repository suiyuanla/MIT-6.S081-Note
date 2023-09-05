# 事前准备

## 一、安装必备软件(arch)

```shell
sudo pacman -S riscv64-linux-gnu-binutils riscv64-linux-gnu-gcc riscv64-linux-gnu-gdb qemu-arch-extra
```
## 二、clone 仓库

```shell
git clone git://g.csail.mit.edu/xv6-labs-2022
# 测试是否能正常编译,如果正常编译则会出现`$`符
make qemu
# Ctrl + c 再迅速按x，退出qemu
```

## 三、配置
> 本人使用的为vscode+c/cpp+clangd，代码分析功能由clangd提供，因此，需要一些额外的配置

1. 安装`bear`用于生成`compile_commands.json`

    ```shell
    sudo pacman -S bear 
    make clean && bear -- make qemu
    ```

2. 编辑`.cland`，用于让`.clangd`找到`bear`生成的json

    ```yaml
    CompileFlags:
    CompilationDatabase: ./
    ```

3. 编辑`.clang-format`，告诉.clangd格式化的需求

    ```yaml
    ---
    Language: Cpp
    BasedOnStyle: Google
    # 这个是重点，禁用头文件自动排序
    SortIncludes: false
    IndentWidth: 2
    ---
    ```
    另外，不要在官方给的代码里格式化
